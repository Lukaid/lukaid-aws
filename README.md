# Repository for AWS

- AWS의 S3, IAM, CloudFront를 활용하여 CI/CD연습

## 1. GitHub Actions란?

<br>

> GitHub Actions는 Github에서 제공하는 배포 서비스이다. GIthub가 MS에 인수되면서 기존의 소스저장소의 기능에서 DevOps플랫폼으로 발전하고 있다.

<br>

## 2. GitHub Actions 사용법

<br>

index.html이 local에서 수정 된 후 repo에 push되면 S3에 index.html이 업로드 되어야 한다. 따라서 GitHub Actions에서 S3에 배포할 수 있는 권한을 줘야한다.

<br>

S3에 업로드 후 CloudFront에도 변경이 되게 하려면 이미 캐싱되어있는 파일을 새롭게 캐싱해야한다. 이러한 것을 invalidation이라고 하며 이 또한 GitHub Actions에서 진행한다.

<br>

따라서 GitHub Actions과 IAM을 연동하여 S3와 CloudFront, 두개의 서비스에 접근 할 권한을 부여해야한다.

<br>

AWS 접속 > IAM 사용자 계정 접속 > 권한 추가 > 기존 정책 직접 연결 > CloudFrontFullAccess 선택 > 권한 추가

<br>

Github에 원하는 이름의 repo를 파서 local의 폴더와 remote시켜준다. 그 후 local의 루트폴더에서 .github/workflows/main.yml을 추가하며 된다. yml은 또 뭔가... 일단은 json같이 구조화된 자료를 담는 파일이라고 생각하자...

코드에디팅에는 pycharm을 사용하였다. 

```yml
name: my-front # 이름, 아무거나
on: # action이 일어날 조건
  push:
    branches:
      - main # main branch가 push 되었을 때
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: 'ap-northeast-2'

    steps:
      - name: Checkout source code.
        uses: actions/checkout@master

      - name: Upload binary to S3 bucket
        uses: jakejarvis/s3-sync-action@master
        with:
          args: --acl public-read --exclude '*' --include 'index.html'
        env:
          AWS_S3_BUCKET: ${{ secrets.BUCKET_NAME }}

      - name: Invalidate cache CloudFront
        uses: chetan/invalidate-cloudfront-action@master
        env:
          DISTRIBUTION: ${{ secrets.DISTRIBUTION_ID }}
          PATHS: '/index.html'
        continue-on-error: true
```

<br>

- secrets.AWS_ACCESS_KEY_ID : IAM 생성 시 발급 받은 값
- secrets.AWS_SECRET_ACCESS_KEY : IAM 생성 시 발급 받은 값
- secrets.BUCKET_NAME : S3의 이름
- secrets.DISTRIBUTION_ID : CloudFront의 배포 ID

딱 봐도 위의 네 값은 그냥 푸시하면 안될 것 처럼 생겼다. 따라서 해당 변수를 따로 저장해주자. github에서 이렇게 보안상 중요한 변수들을 따로 저장 할 수 있게 마련해두었다.

해당 레포로 돌아가서 Settings > Secrets > New repository secret클릭 후 위의 변수 명을 name에 해당하는 값을 value에 넣고 Add secret을 눌러준다.