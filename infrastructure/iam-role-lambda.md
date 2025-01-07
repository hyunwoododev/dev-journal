# Bitbucket Pipelines에서 AWS IAM Role을 사용하여 Lambda 함수 업데이트하기 (OIDC 사용)

이 문서는 Bitbucket Pipelines에서 OpenID Connect(OIDC)를 사용하여 AWS IAM Role을 연결하고, 안전하게 Lambda 함수를 업데이트하는 방법을 단계별로 설명합니다.

**전체 프로세스:**

1.  **AWS IAM에서 Identity Provider (IdP) 생성:** Bitbucket Pipelines를 신뢰할 수 있는 IdP로 등록합니다.
2.  **AWS IAM에서 Role 생성:** Bitbucket Pipelines가 맡을 역할(Role)을 생성하고, 필요한 권한을 부여합니다.
3.  **Bitbucket Pipelines 설정:** `bitbucket-pipelines.yml` 파일에서 OIDC를 활성화하고, 생성한 IAM Role을 사용하도록 설정합니다.

---

## 1. AWS IAM에서 Identity Provider (IdP) 생성

**목표:** Bitbucket Pipelines를 AWS 계정에서 신뢰할 수 있는 OpenID Connect Identity Provider로 등록합니다.

**단계:**

1.  AWS Management Console에 로그인하고 **IAM** 서비스로 이동합니다.
2.  왼쪽 메뉴에서 **"Identity providers"** 를 선택하고 **"Add provider"** 를 클릭합니다.
3.  **"Provider type"** 으로 **"OpenID Connect"** 를 선택합니다.
4.  **"Provider URL"** 에 `https://api.bitbucket.org/2.0/workspaces/{workspace_id}/pipelines-config/identity/oidc` 를 입력합니다. **`{workspace_id}`** 부분을 자신의 Bitbucket Workspace ID로 바꿔야 합니다. Workspace ID는 Bitbucket 저장소 URL에서 확인할 수 있습니다. (예: `https://bitbucket.org/`**`{workspace_id}`**`/repository`)
5.  **"Audience"** 에 `ari:cloud:bitbucket::workspace/{workspace_id}` 를 입력합니다. 역시 **`{workspace_id}`** 부분을 자신의 Bitbucket Workspace ID로 바꿔야 합니다.
6.  **"Get thumbprint"** 를 클릭하여 Bitbucket의 인증서 지문을 가져옵니다.
7.  **"Add provider"** 를 클릭하여 IdP를 생성합니다.

## 2. AWS IAM에서 Role 생성

**목표:** Bitbucket Pipelines가 AWS 리소스에 접근할 때 사용할 IAM Role을 생성하고, Lambda 함수 업데이트에 필요한 권한을 부여합니다.

**단계:**

1.  AWS Management Console에서 **IAM** 서비스로 이동합니다.
2.  왼쪽 메뉴에서 **"Roles"** 를 선택하고 **"Create role"** 을 클릭합니다.
3.  **"Trusted entity type"** 으로 **"Web identity"** 를 선택합니다.
4.  **"Identity provider"** 드롭다운 메뉴에서 1단계에서 생성한 IdP (예: `api.bitbucket.org`)를 선택합니다.
5.  **"Audience"** 는 자동으로 1단계에서 설정한 Audience 값으로 채워질 것입니다.
6.  **"Next"** 를 클릭합니다.
7.  **"Permissions policies"** 섹션에서 **"Create policy"** 를 클릭하여 새 브라우저 탭을 엽니다. (기존에 만들어둔 정책이 있다면 사용해도 무방합니다.)
8.  **"JSON"** 탭을 선택하고 아래 정책을 붙여넣습니다.

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "UpdateLambdaFunctionCode",
                "Effect": "Allow",
                "Action": "lambda:UpdateFunctionCode",
                "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME"
            }
        ]
    }
    ```

    *   **`REGION`**: Lambda 함수가 위치한 리전을 입력합니다 (예: `us-east-1`).
    *   **`ACCOUNT_ID`**: AWS 계정 ID를 입력합니다.
    *   **`FUNCTION_NAME`**: 업데이트하려는 Lambda 함수의 이름을 입력합니다.

9.  **"Review policy"** 를 클릭합니다.
10. **"Name"** 에 정책 이름 (예: `BitbucketPipelinesLambdaUpdatePolicy`)을 입력하고 **"Create policy"** 를 클릭합니다.
11. 다시 Role 생성 탭으로 돌아와서, 방금 생성한 정책을 검색하여 선택합니다.
12. **"Next"** 를 클릭합니다.
13. **"Role name"** 을 입력합니다 (예: `BitbucketPipelinesLambdaUpdateRole`).
14. **"Create role"** 을 클릭하여 Role을 생성합니다.
15. 생성된 Role의 요약 페이지에서 **Role ARN** 을 복사해 둡니다. (Bitbucket Pipelines 설정에 필요합니다.)

## 3. Bitbucket Pipelines 설정

**목표:** Bitbucket Pipelines가 OIDC를 사용하여 AWS에 인증하고, 생성한 IAM Role을 사용하여 Lambda 함수를 업데이트하도록 설정합니다.

**단계:**

1.  Bitbucket 저장소의 루트 디렉토리에 `bitbucket-pipelines.yml` 파일을 생성하거나 엽니다.
2.  다음 내용을 `bitbucket-pipelines.yml` 파일에 추가합니다.

    ```yaml
    pipelines:
      default:
        - step:
            oidc: true
            script:
              - export AWS_REGION="YOUR_LAMBDA_REGION"
              - export AWS_ROLE_ARN="arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/YOUR_BITBUCKET_PIPELINES_ROLE"
              - pipe: atlassian/aws-cli:1.6.0
                variables:
                  AWS_REGION: $AWS_REGION
                  AWS_ROLE_ARN: $AWS_ROLE_ARN
                  COMMAND: 'lambda update-function-code'
                  # Add extra arguments to the aws command below, e.g.
                  ARGS: '--function-name YOUR_LAMBDA_FUNCTION_NAME --zip-file fileb://deployment.zip'
    ```

    *   **`oidc: true`**: Bitbucket Pipelines가 OIDC를 사용하여 AWS와 인증하도록 설정합니다.
    *   **`YOUR_LAMBDA_REGION`**: Lambda 함수가 배포된 리전으로 변경합니다 (예: `us-east-1`).
    *   **`arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/YOUR_BITBUCKET_PIPELINES_ROLE`**: 2단계에서 생성한 IAM Role의 ARN으로 변경합니다.
    *   **`YOUR_AWS_ACCOUNT_ID`**: AWS 계정 ID로 변경합니다.
    *   **`YOUR_BITBUCKET_PIPELINES_ROLE`**: 2단계에서 생성한 IAM Role 이름으로 변경합니다.
    *   **`YOUR_LAMBDA_FUNCTION_NAME`**: 업데이트하려는 Lambda 함수의 이름으로 변경합니다.
    *   **`--zip-file fileb://deployment.zip`**:  `deployment.zip`은 빌드 과정에서 생성된 Lambda 함수의 코드 패키지 파일명으로, 상황에 맞게 수정해야 합니다.

3.  변경 사항을 커밋하고 Bitbucket 저장소에 푸시합니다.

## 참고

*   이 설정은 Bitbucket Pipelines에서 AWS CLI를 사용하는 예시입니다. 필요에 따라 다른 도구나 스크립트를 사용할 수 있습니다.
*   `deployment.zip` 파일은 빌드 과정에서 생성되어야 합니다. `bitbucket-pipelines.yml` 파일에서 빌드 단계를 추가하여 `deployment.zip` 파일을 생성하는 로직을 포함해야 합니다.
*   보안을 위해, Lambda 함수를 업데이트하는 데 필요한 최소한의 권한만 IAM Role에 부여하는 것이 좋습니다.
*   더 자세한 정보는 Bitbucket Pipelines 및 AWS IAM 문서를 참조하십시오.

---

이 문서가 Bitbucket Pipelines에서 AWS IAM Role을 사용하여 Lambda 함수를 안전하게 업데이트하는 데 도움이 되었기를 바랍니다.
