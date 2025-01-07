# Bitbucket Pipelines에서 AWS Secrets Manager를 사용하여 Lambda 함수 배포하기

이 문서는 Bitbucket Pipelines에서 AWS Secrets Manager에 저장된 Lambda 실행 역할(Role) ARN을 가져와 `atlassian/aws-lambda-deploy` 파이프를 사용하여 Lambda 함수를 안전하게 배포하는 방법을 설명합니다.

**목표:**

*   Lambda 함수의 실행 역할 ARN을 AWS Secrets Manager에 안전하게 저장합니다.
*   Bitbucket Pipelines에서 Secrets Manager에 접근하여 Role ARN을 가져옵니다.
*   `atlassian/aws-lambda-deploy` 파이프의 `ROLE_ARN` 변수를 사용하여 Lambda 함수 배포 시 실행 역할을 지정합니다.

**사전 요구 사항:**

*   AWS 계정
*   Bitbucket 계정 및 저장소
*   Lambda 함수에 연결할 IAM Role 생성
*   Bitbucket Pipelines가 AWS에 인증할 수 있도록 OpenID Connect (OIDC) 설정 및 IAM Role 생성 (자세한 내용은 [이 문서](iam-role.md) 참조)

**단계:**

## 1. AWS Secrets Manager에 Lambda Role ARN 저장

1.  AWS Management Console에서 **Secrets Manager** 서비스로 이동합니다.
2.  **"Store a new secret"** 을 클릭합니다.
3.  **"Secret type"** 으로 **"Other type of secret"** 을 선택합니다.
4.  **"Key/value pairs"** 탭에서:
    *   **Key:** `lambda_role_arn` (또는 원하는 키 이름)
    *   **Value:** Lambda 함수에 연결할 IAM Role의 ARN을 입력합니다.
5.  **"Next"** 를 클릭합니다.
6.  **"Secret name"** 을 입력합니다 (예: `bitbucket/lambda-role-arn`). **이 이름을 기억해야 합니다.**
7.  나머지 설정은 기본값으로 두고, **"Next"** 를 클릭합니다.
8.  **"Review"** 페이지에서 설정을 확인하고 **"Store"** 를 클릭하여 Secret을 생성합니다.

## 2. Bitbucket Pipelines IAM Role에 Secrets Manager 접근 권한 부여

Bitbucket Pipelines가 맡게 될 IAM Role에 Secrets Manager에서 Secret을 읽을 수 있는 권한을 부여합니다.

1.  AWS Management Console에서 **IAM** 서비스로 이동합니다.
2.  왼쪽 메뉴에서 **"Roles"** 를 선택하고, Bitbucket Pipelines가 사용할 IAM Role (예: `BitbucketPipelinesLambdaUpdateRole`)을 찾아서 클릭합니다.
3.  **"Permissions"** 탭에서 **"Add permissions" -> "Attach policies"** 를 클릭합니다.
4.  **"Filter policies"** 에서 `SecretsManager` 를 검색합니다.
5.  상황에 맞게 `SecretsManagerReadWrite` 혹은 `SecretsManagerRead` 중 선택합니다. **최소 권한 원칙에 따라, `SecretsManagerReadWrite` 보다는 `SecretsManagerRead`를 사용하는 것이 좋습니다.**
6.  또는, **"Create policy"** 를 클릭하여 세분화된 권한 (특정 Secret에 대한 읽기 권한만 부여)을 가진 사용자 지정 정책을 생성할 수 있습니다. 아래는 사용자 지정 정책 예시입니다:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "ReadLambdaRoleArnFromSecretsManager",
                "Effect": "Allow",
                "Action": [
                    "secretsmanager:GetSecretValue"
                ],
                "Resource": [
                    "arn:aws:secretsmanager:REGION:ACCOUNT_ID:secret:SECRET_NAME*"
                ]
            }
        ]
    }
    ```

    *   `REGION`을 Secrets Manager Secret이 저장된 리전으로 변경합니다.
    *   `ACCOUNT_ID`를 AWS 계정 ID로 변경합니다.
    *   `SECRET_NAME`을 1단계에서 생성한 Secret 이름으로 변경합니다 (예: `bitbucket/lambda-role-arn`).
7.  **"Attach policies"** 를 클릭하여 정책을 Role에 연결합니다.

## 3. `bitbucket-pipelines.yml` 파일 구성

```yaml
image: golang:1.23.3

pipelines:
  branches:
    main:
      - step:
          name: Build and Deploy to Lambda
          oidc: true
          script:
            - export AWS_REGION="YOUR_LAMBDA_REGION" # Lambda 함수가 배포된 리전
            - export AWS_ROLE_ARN="arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/YOUR_BITBUCKET_PIPELINES_ROLE" # Bitbucket Pipelines가 맡을 IAM Role
            - export SECRETS_MANAGER_SECRET_NAME="bitbucket/lambda-role-arn" # Secrets Manager에 저장된 Secret 이름
            - export LAMBDA_FUNCTION_NAME="YOUR_LAMBDA_FUNCTION_NAME" # Lambda 함수 이름

            - apt-get update && apt-get install -y zip jq

            # Secrets Manager에서 Lambda Role ARN 가져오기
            - export LAMBDA_ROLE_ARN=$(aws secretsmanager get-secret-value --secret-id $SECRETS_MANAGER_SECRET_NAME --region $AWS_REGION --query SecretString --output text | jq -r '.lambda_role_arn')

            - GOOS=linux GOARCH=arm64 go build -tags lambda.norpc -o bootstrap main.go
            - zip deployment.zip bootstrap

            - pipe: atlassian/aws-lambda-deploy:1.12.0
              variables:
                AWS_REGION: $AWS_REGION
                AWS_ROLE_ARN: $AWS_ROLE_ARN # Bitbucket Pipelines가 맡을 IAM Role
                FUNCTION_NAME: $LAMBDA_FUNCTION_NAME
                ZIP_FILE: 'deployment.zip'
                ROLE_ARN: $LAMBDA_ROLE_ARN # Secrets Manager에서 가져온 Lambda Role ARN
