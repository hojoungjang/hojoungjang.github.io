---
layout: post
title:  "Github Actions 에서 AWS 시크릿 더 안전하게 사용"
date:   2025-05-25 12:00:00 +0900
categories: [AWS, Github Actions]
tags: []
---

코드 작성에서 나아가 애플리케이션을 빌드하고 배포 할때 Github Actions 를 사용하여 작업을 간편하게 자동화 할 수 있습니다.
Github Actions 의 강점은 커뮤니티가 정말 다양한 오픈된 라이브러리를 제공하고 있다는 점인데요. 요즘은 애플리케이션을 
클라우드 서비스 업체를 통해 배포하고 호스팅하는 유즈케이스가 많다보니 자연스럽게 클라우드 업체 또는 커뮤니티에서
사용가능한 Github Actions 솔루션을 제공합니다.

그 중 현시점 가장 보편적으로 사용되는 클라우드 서비스 업체인 AWS 의 서비스를 사용하기 위해 액션에서
인증 (로그인) 을 받는 간단한 방법과 더 나아가 권장되는 임시 인증 정보를 통한 인증을 살펴보려 합니다. Github Actions 에서 AWS 인증을 받는 방법으로 [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials) 액션을 사용할 수 있습니다. 아래에서 두가지 방법으로 이 액션을 통해 인증 받는 방법을 보고 비교해 보겠습니다.

## Access Key ID 와 Secret Access Key 사용
먼저 가장 간단한 방법은 access key ID 와 secret access key 를 사용한 방법입니다.

![alt text](/assets/images/2025-05-25-naive-way-diagram.png)

이 방법은 마치 아이디와 비밀번호를 통한 인증입니다. 먼저 AWS IAM 에서 유저를 생성하고 유저에게 액션에서 사용하는 AWS 작업에 필요한 권한을 적절히 부여해줍니다. 그리고 유저에 대해 access key 를 생성하여 access key 와 secret access key 값을 Github 에 secret 변수로 등록하고 액션에서 읽어와서 필요한 AWS 서비스 작업에 넘겨주어 인증하도록 할 수 있습니다.

간단하게로는 [IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) 은 인증과 인가를 담당하는 서비스이고 AWS 서비스 계정안에 특정 권한 (예를 들어 S3 서비스에 대한 버킷 생성, 변경, 삭제 등) 을 가진 사용자를 만들어 그 사용자의 아이디와 비밀번호를 이용하여 액션안에 작업이 본인의 AWS 서비스에 접근 (예를 들어 압축한 소스코드를 S3 에 업로드) 하기 위해  로그인 하도록 도와줍니다.

그리고 secret 변수 등록은 Github 리포지토리 설정에 들어가여 등록이 가능합니다. 기본적으로 인증정보는 코드와 같이 관리하는 것은 외부에 노출되어 보안에 위배되는 행동이기 때문에 Github 에서는 이런식으로 리포지토리에 설정해두고 액션에서 사용가능하도록 해줍니다. [문서](https://docs.github.com/ko/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)를 통해 더 자세히 보실 수 있습니다.

아래는 access key 와 secret access key 를 통해 인증을 받는 액션 step 작성 예시 입니다.
```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4.1.0
    with:
      aws-access-key-id: {% raw %}${{ secrets.AWS_ACCESS_KEY_ID }}{% endraw %}
      aws-secret-access-key: {% raw %}${{ secrets.AWS_SECRET_ACCESS_KEY }}{% endraw %}
      aws-region: us-east-2
      role-to-assume: {% raw %}${{ secrets.AWS_ROLE_TO_ASSUME }}{% endraw %}
      role-external-id: {% raw %}${{ secrets.AWS_ROLE_EXTERNAL_ID }}{% endraw %}
      role-duration-seconds: 1200
      role-session-name: MySessionName
```
이런식으로 리포지토리 시크릿에 값들을 저장하고 액션 수행시 저장된 credential 값들을 직접 읽어와 파라미터로 넘겨줍니다.

## Github OIDC 를 통한 인증
위에 방법은 직관적이고 편리하지만 보안적인 면에서는 문제가 될 여지가 있습니다. 인증정보를 외부 시스템에 보관하게 되고 인증정보는 장기적으로 유효하기 때문에 노출되었을 경우 의도하지 않은 사용자가 서비스에 접근이 가능합니다. 또 추가로 이러한 문제의 여지 때문에 주기적으로 인증정보 즉 access key 와 secret key 를 변경하도록 권장됩니다. 마치 저희의 이메일 또는 기타 웹서비스의 로그인 아이디와 비번을 주기적으로 변경해주면 더 계정 보안에 도움되는 것 처럼 말이죠. 헌데 이건 매우 귀찮고 번거롭습니다.

사실 조심한다면 인증 키가 노출되는 사고가 일어날 일은 희박하지만 문제가 될 여지가 있다는건 변하지 않습니다. 만약 이런 문제를 몇가지 간단한 설정으로 피할 수 있다면 그렇게 하지 않을까요? 그렇게 할 수 있는 방법이 Github OIDC 를 통한 인증을 사용하는 것 입니다. 이 방법은 설정이 크게 복잡하지 않고 별도로 추가 비용이 발생하지 않았습니다.

### OIDC 란?
먼저 OIDC 는 무엇일까요? OpenID Connect 의 약자로 OIDC 는 OAuth 2.0 를 기반으로 동작하고 OAuth 프레임워크 위에 신분 (identity) 정보가 문맥에 추가되어 있습니다. 

OAuth 는 인가에 초점을 두고 있습니다. 클라이언트 앱이 특정 리소스에 대한 접근을 리소스 오너로 부터 권한을 받아 동작합니다. OIDC 도 OAuth 처럼 클라이언트, 리소스 오너, 리소스 서버, 인증 서버 간 거의 동일한 상호작용이 일어나지만 최종 목적은 리소스 오너의 identity 정보를 받아오는 것 입니다. 즉 인증에 초점을 두고 있습니다.

### Github Actions 작업이 OIDC 를 이용하여 AWS 인증 받는 과정
그렇다면 이제 OIDC 가 어떻게 Github Actions 작업 안에서 AWS 로 부터 인증 받기위해 동작하는지 알아보겠습니다. 

![alt text](/assets/images/2025-05-25-oidc-way-diagram.png)

1. 먼저 액션 안에서 Github OIDC provider 에 해당 작업에 대한 아이덴티티 토큰 (JWT) 을 요청합니다. 이 작업은 [공식문서](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect#updating-your-actions-for-oidc)에 정리되었듯이 해당 액션 작업에 부여된 고유한 토큰 (`ACTIONS_ID_TOKEN_REQUEST_TOKEN`) 을 통해 작업의 아이덴티티 토큰을 Github OIDC 에 요청 할 수 있습니다.
2. 응답으로 돌려 받은 아이덴티티 토큰과 원하는 AWS IAM role 을 AWS (즉 리소스 서버) 에 요청합니다.
3. AWS 에서 전달 받은 요청에 대해 아이덴티티를 검증하고 유효하면 요청한 role 에 대한 credential 을 돌려줍니다.
4. 이제 응답받은 credential 을 이용해 추후 액션 작업에서 원하는 AWS 서비스를 사용 할 수 있습니다.

### 자세한 설정 방법
설정 방법은 저보다도 공식문서에서 이미 잘 다루고 있어서 간단히 요약해서 정리해보았습니다. 각 단계별로 공식문서 자료를 참고하시면 도움이 될 것 같습니다.

1. AWS 콘솔을 이용한 방법입니다. AWS IAM 서비스로 들어가 왼쪽 "Identity providers" 메뉴에 들어가 Github OIDC를 새로운 provider 로 추가해주세요 ([공식 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html#manage-oidc-provider-console)). 이 부분이 바로 AWS 가 Github OIDC 서비스를 Identity provider 로 인정할 수 있도록 설정하는 부분 입니다.

2. 다시 IAM 서비스 콘솔에서 이번에는 "Roles" 메뉴에 들어가 새로운 role 을 추가해줍니다. 자세한건 [공식 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html#idp_oidc_Create_GitHub)를 참고해주세요.
    1. 엔티티 타입은 "Web identity" 그리고 Identity provider 는 1번에서 추가한 Github 으로 골라주세요.
    2. 다음 단계에서 assume role 을 통해 부여받을 권한을 골라주세요.
    3. 설정을 검토하고 만들어주세요.

3. 이제 액션 yaml 파일을 열어 `aws-actions/configure-aws-credentials` 액션을 사용하여 인증절차를 액션에서 OIDC 를 통한 인증 절차를 하도록 해주세요. 아래는 샘플 예시 입니다. 자세한건 [공식 문서](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#updating-your-github-actions-workflow)를 참고해주세요.

    ```yaml
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        id: aws_credentials
        with:
          aws-region: # 여기에 사용하고 있는 AWS 서비스 지역
          role-to-assume: # 여기에 2번에서 만든 role 의 ARN
    ```
    여기서는 꼭 `id-token: write` 설정을 넣어주어야 Github OIDC 로부터 현재 액션 작업의 아이덴티티 토큰을 받아 컨텍스트 저장할 수 있습니다. 만약 `steps` 에 [`actions/checkout`](https://github.com/actions/checkout) 을 사용하고 있다면 `contents: read` 도 포함시켜주어야 저장소 checkout 에 필요한 권한을 부여할 수 있습니다.

## 마무리
개인 프로젝트를 AWS 에 배포하며 적용해보았던 Github OIDC provider 를 이용한 Github Actions 안애서 AWS 서비스 사용에 대해 정리해보았습니다. 보편적으로는 간단하게 access key 와 secret key 를 이용한 방법이 많이 알려져 있는것 같아 글로 정리하여 보안적으로 더 개선된 방법을 공유해보고 싶었습니다.

작은 프로젝트에서는 크게 상관 없을듯 하지만 AWS 안에서 관리하고 사용하는 키가 많은 규모있는 프로젝트에서는 주기적으로 키를 변경 (rotate) 하는 작업은 쉽지 않을 것 같습니다. 한번의 설정으로 보안적인 장점과 키 변경같은 추가 작업을 유지보수 프로세스에서 제거 할 수 있기 때문에 충분히 의미있는 기능인 것 같습니다.
