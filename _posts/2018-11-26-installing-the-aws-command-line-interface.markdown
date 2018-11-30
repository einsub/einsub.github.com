---
layout: post
title: "Amazon CLI로 ECS에 서버 띄우기 3 - AWS CLI 설치"
date: 2018-11-26 21:50:16
author: Reid
categories:
  - engineering
tags:
  - amazon
  - aws
  - ecs
  - docker
  - amazon cli
published: true
---
# AWS CLI
AWS CLI는 AWS의 각종 기능을 terminal에서 명령어만으로 실행 할 수 있게 해줍니다. 직접 AWS 콘솔 페이지에서 버튼을 눌러 실행하는 것 보다 간편화며, 이후 스크립트로 만들어 구동하는 것도 쉬워집니다.

> 참고: AWS CLI는 새로 추가되는 서비스들에 대한 업데이트가 빈번하기 때문에 [Release note](https://github.com/aws/aws-cli/releases)를 자주 살펴보면서 최신 버전을 유지해주세요.

## 설치
### Python 설치

> 참고: 글을 작성중인 환경은 macOS Mojave 10.14.1이며, Python 2.7.15와 pip 18.0이 기본으로 설치되어 있습니다.

AWS CLI는 파이썬으로 작성되었습니다. Python 2.6.4 이상 혹은 Python 3.3 이상 버전을 설치하세요. [홈페이지](https://www.python.org/downloads/)에서 직접 설치하거나 mac이라면 Homebrew를 이용합니다.
```
$ brew install python
```
Python 패키지 관리자인 pip도 설치합니다.
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user
```

### AWS CLI 설치
pip으로 AWS CLI를 설치합니다.
```shell
$ pip install awscli --upgrade --user
```
- --upgrade: 이미 설치되어 있다면 필요에 따라 파일들을 업그레이드시킵니다.
- --user: OS의 user 디렉토리 하위에 설치하기 때문에 global로 설치한 프로그램과의 충돌을 방지해줍니다. root 권한이 필요없기 때문에 sudo를 붙여 줄 필요도 없습니다. 다만, 어디서나 실행 가능하도록 path 설정을 해줘야 합니다.

어디서든 AWS CLI를 실행 할 수 있도록 path를 설정합니다. aws 설치 중에 나오는 메시지의 마지막 부분을 보시면 설치 된 경로가 어디고, 그 경로가 PATH에 추가되어 있지 않으니 추가하라는 친절한 메시지가 나옵니다. 그 경로를 PATH에 추가합니다.

`The scripts pyrsa-decrypt, pyrsa-decrypt-bigfile, pyrsa-encrypt, pyrsa-encrypt-bigfile, pyrsa-keygen, pyrsa-priv2pub, pyrsa-sign and pyrsa-verify are installed in '/Users/leeinsub/Library/Python/2.7/bin' which is not on PATH.
 Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.`

.bash_profile 끝에 다음 한 줄을 추가합니다.

`export PATH=/Users/leeinsub/Library/Python/2.7/bin:${PATH}`

설치가 잘 되었는지 version 정보를 확인합니다.
```bash
$ aws --version
aws-cli/1.16.60 Python/2.7.10 Darwin/18.2.0 botocore/1.12.50
```
## 설정
AWS CLI를 설정하기 위해 `aws configure` 명령을 입력합니다.
```bash
$ aws configure
AWS Access Key ID [None]: 본인의 Access Key ID
AWS Secret Access Key [None]: 본인의 Secret Access Key
Default region name [None]: ap-northeast-2
Default output format [None]: json
```
IAM 사용자를 만들면 부여되는 액세스 키와 보안 키를 입력합니다. 기본 리전 이름으로는 서울인 ap-northeast-2를 입력합니다.