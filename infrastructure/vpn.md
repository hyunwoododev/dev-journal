
# VPN 설정 가이드 (OpenVPN 및 AWS Client VPN)

이 가이드는 OpenVPN과 AWS Client VPN을 사용하여 안전한 VPN 연결을 설정하는 과정에 대한 자세한 설명을 제공합니다. 
각 단계에서 수행되는 작업과 그 의미를 포함하여 전체 과정을 설명합니다.

## 개요

이 설정은 AWS Client VPN을 사용하여 VPN 엔드포인트를 생성하고, Easy-RSA를 통해 필요한 인증서를 생성하며, 클라이언트가 안전하게 VPC 내 리소스에 접근할 수 있도록 구성하는 과정입니다.

---

## 1. 필수 라이브러리 준비

### 1.1 Easy-RSA 저장소 클론

Easy-RSA는 VPN 서버와 클라이언트를 인증하기 위한 인증서를 생성하는 도구입니다.

```bash
# Easy-RSA 저장소를 클론합니다.
git clone https://github.com/OpenVPN/easy-rsa.git

# Easy-RSA 디렉터리로 이동합니다.
cd easy-rsa/easyrsa3
```

### 1.2 PKI와 인증서 이해
   - **PKI (공개 키 인프라)**: 안전한 통신을 위해 인증서를 생성하고 관리하는 시스템.
   - **CA (인증 기관)**: 루트 인증서로, 서버와 클라이언트 인증서를 서명하여 신뢰 체인을 형성합니다.
   - **서버 인증서**: 클라이언트가 서버의 신뢰성을 확인할 수 있도록 해줍니다.
   - **클라이언트 인증서**: VPN에 연결하려는 개별 클라이언트를 인증합니다.

---

## 2. 인증서 생성 및 관리 (터미널 명령어 사용)

### 2.1 PKI 환경 초기화

```bash
./easyrsa init-pki
```
   - PKI 환경을 초기화하여 인증서 관리에 필요한 폴더 구조를 설정합니다.

### 2.2 CA (인증 기관) 인증서 생성

```bash
./easyrsa build-ca nopass
```
   - CA 인증서를 생성하여 서버와 클라이언트 인증서를 서명합니다.
   - **CA 이름**을 입력합니다 (예: `auto-vpn`). 이 이름은 CA를 식별하는 데 사용됩니다.

### 2.3 서버 인증서 생성

```bash
./easyrsa build-server-full <서버 이름> nopass
```
   - `<서버 이름>`을 `auto-vpn.com`과 같은 도메인 형식으로 입력합니다.
   - VPN 서버에 사용할 인증서와 프라이빗 키를 생성합니다.
   - 생성 후 "yes"를 입력하여 인증서 서명을 확인합니다.

### 2.4 클라이언트 인증서 생성 (선택 사항)

```bash
./easyrsa build-client-full <클라이언트 이름> nopass
```
   - `<클라이언트 이름>`을 `user1.client.com` 등으로 지정합니다.
   - 클라이언트의 인증서와 프라이빗 키를 생성합니다.
   - 생성 후 "yes"를 입력하여 인증서 서명을 확인합니다.

### 2.5 인증서 파일 정리

새 폴더를 만들어 필요한 인증서와 키 파일을 정리하여 관리합니다.

아래 파일들을 새 폴더에 복사합니다:

- **`ca.crt`**: CA 인증서
- **서버 인증서 및 프라이빗 키**: `pki/issued/` 및 `pki/private/` 경로에 있음
- **클라이언트 인증서 및 프라이빗 키** (옵션): `pki/issued/` 및 `pki/private/` 경로에 있음

---

## 3. AWS 설정

### 3.1 인증서 AWS Certificate Manager (ACM)에 등록

1. **ACM으로 이동**하여 **인증서 가져오기**를 선택합니다.
2. 각 필드에 인증서를 입력합니다:
   - **Certificate body**: 서버의 `.crt` 파일 내용
   - **Certificate private key**: 서버의 `.key` 파일 내용
   - **Certificate chain**: `ca.crt` 파일 내용

3. 클라이언트 인증서에 대해서도 동일한 작업을 수행하여 ACM에 두 개의 인증서를 등록합니다.

> 이 단계가 완료되면 ACM 인증서 목록에 서버와 클라이언트 인증서가 추가되어 있어야 합니다.

### 3.2 Client VPN 엔드포인트 설정

1. **VPC** > **Client VPN Endpoints** > **Create Client VPN Endpoint**로 이동합니다.
2. 아래와 같이 설정합니다:
   - **Name tag**: `auto-vpn-endpoint`
   - **Client CIDR**: `100.64.0.0/22` (VPN 클라이언트에게 할당할 IP 범위)
   - **Authentication**: Mutual Authentication을 선택하고 서버와 클라이언트 인증서 ARN을 설정합니다.
   - **Enable split-tunnel**: 활성화하여 트래픽이 필요한 리소스에만 VPN을 통해 접근하도록 설정합니다.

3. **VPC 연결**: 원하는 **VPC ID**를 연결합니다.

### 3.3 서브넷 연결 설정

- 생성한 VPN 엔드포인트를 선택합니다.
- **Target Network Associations** 탭으로 이동하여 **Associate target network**를 클릭합니다.
- 프라이빗 서브넷을 선택하여 연결합니다.

### 3.4 접근 규칙 설정 (Authorization Rules)

- VPC의 **CIDR 블록**을 지정하여 VPN 클라이언트가 접근할 수 있는 IP 대역을 정의합니다.

---

## 4. 클라이언트 구성 설정

1. **클라이언트 구성 다운로드**: VPN 엔드포인트 페이지에서 **Client configuration** 파일을 다운로드합니다.
2. **.ovpn 파일 수정**: 파일을 열고 아래 내용을 추가합니다.

   ```
   <cert>
   ...(클라이언트 .crt 파일 내용)...
   </cert>
   <key>
   ...(클라이언트 프라이빗 키 파일 내용)...
   </key>

   reneg-sec 0
   ```

---

## 5. AWS VPN 클라이언트 다운로드 및 연결

### 5.1 AWS VPN 클라이언트 다운로드
- [AWS Client VPN Download](https://aws.amazon.com/vpn/client-vpn-download/) 페이지에서 AWS VPN 클라이언트를 다운로드하고 설치합니다.

### 5.2 프로필 추가
- AWS VPN 클라이언트를 열고 **파일 > 프로필 관리**로 이동하여 `.ovpn` 파일을 추가합니다.

### 5.3 VPN 연결
- 프로필을 선택하고 **Connect** 버튼을 눌러 VPN에 연결합니다.

---

## 요약

이 가이드는 AWS Client VPN과 OpenVPN을 사용하여 안전하게 VPC 리소스에 접근할 수 있도록 설정하는 방법을 제공합니다. 
서버와 클라이언트 인증서를 통해 상호 인증을 설정하고, split-tunneling을 통해 필요한 트래픽만 VPN을 통해 전달하여 보안성과 효율성을 동시에 확보할 수 있습니다.
