## Provisioning

**프로비저닝(Provisioning)**은 사용자 또는 비즈니스의 요구사항에 따라 미리 시스템 자원을 할당, 배치, 배포해두었다가 즉시 사용할 수 있도록 준비하는 절차를 말한다. 어떤 시스템을 제공하느냐에 따라 아래와 같이 프로비저닝을 분류할 수 있다.

<br>

- 서버 자원 프로비저닝
- OS 프로비저닝
- 소프트웨어 프로비저닝
- 스토리지 프로비저닝
- 계정 프로비저닝

<br>

## AWS EC2

EC2 인스턴스에 Django 프로젝트를 배포해보자.

이번에 사용한 AMI는 `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type`이고, IAM을 통해 계정까지 생성했다는 전제하에 진행할 예정이다.

.pem 파일은 `~/.ssh` 디렉토리에 저장하는 것을 권장하고, `ssh` 커맨드를 통해 원격으로 접속한다.

<br>

> **$ ssh -i [키페어 경로 + .pem 파일명] [계정명]@퍼블릭DNS주소**
>
> 또는
>
> **$ ssh -i [키페어 경로 + .pem 파일명] [계정명]@퍼블릭IP주소**

<br>

Ubuntu AMI 인스턴스의 경우 디폴트 계정명은 `ubuntu`이다.

ssh 프로토콜로 어느 원격지를 접속하든 아래 메시지는 당연히 뜨는데, 이번 경우에는 에러가 발생한다.

<br>

> *Are you sure you want to continue connectiong (yes/no) ?* **yes**

<br>

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for '/home/user/.ssh/django-board.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/home/user/.ssh/django-board.pem": bad permissions
Permission denied (publickey).
```

.pem 파일에 대한 read 권한이 막혀 있기 때문이다. 권한 변경 후 재시도하면 접속된다.

<br>

> **$ chmod 400 django-board.pem**

<br>

### 서버 기본 설정

- locale 설정

<br>

> **$ sudo vi /etc/default/locale**

<br>

```D
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
```

작성 후 재접속한다.

<br>

- 패키지 정보 업데이트

> **$ sudo apt-get -y update**

<br>

- 패키지 의존성 검사 및 업그레이드

> **$ sudo apt-get -y dist-upgrade**
>
> *엔터, 엔터*

<br>

- pip 설치

> **$ sudo apt-get install python-pip**

<br>

- zsh 및 ohmyzsh 설치

> **$ sudo apt-get install zsh**
>
> **$ sudo curl -L http://install.ohmyz.sh | sh**
>
> **$ sudo chsh ubuntu -s /usr/bin/zsh**

<br>

서버 기본 설정은 완료했다. 이제 Python, Django 환경 설정을 해주고 프로젝트 소스를 가져오자. 필자는 git clone으로 불러 왔다.

<br>

### 이슈 발생

분명 EC2 콘솔에서 인바운드 규칙에 8000번 포트도 추가해줬는데 아무리 접속해도 프로젝트가 뜨지 않았다. 헤매고 헤매다 은지님의 도움으로 `settings.py`에 문제가 있음을 알아냈다.

`ALLOWED_HOSTS`에 내 EC2 퍼블릭 IP 또는 퍼블릭 DNS 주소를 넣어서 액세스를 허용해주어야 했다.

<br>

```python
ALLOWED_HOSTS = ['127.0.0.1', 'localhost', '54.180.93.63']
```

<br>

그러나! 아직 접속이 안된다.

`./manage.py runserver`를 하면 서버 구동 메시지 중에 `127.0.0.1:8000` 어쩌고를 확인할 수 있는데 `127.0.0.1`이라는 IP로만 접속할 수 있다는 뜻이다. 이걸 해결하려면 커맨드를 약간 수정해야 한다.

<br>

> **$ ./manage.py runserver 0:8000**

<br>

성공!

<br>

## AWS S3

AWS 콘솔에서 직접 S3 버킷을 생성할 수도 있지만 파이썬의 경우 `boto3` SDK를 통해 생성하는 방법 또한 존재한다.

시작하기 전에, 먼저 여러 AWS 서비스들을 제어하거나 자동화할 수 있는 CLI를 설치하자.

<br>

### AWS CLI 설치

<br>

> **$ pip install awscli**
>
> 정상적으로 설치되었는지 확인
>
> **$ aws --version**
>
> *aws-cli/1.16.296 Python/3.7.5 Darwin/18.7.0 botocore/1.13.32*

<br>

### AWS CLI 계정 설정

<br>

> **$ aws configure**
>
> *AWS Access Key ID [None]: **공개키 입력***
>
> *AWS Secret Access Key [None]: **비밀키 입력***
>
> *Default region name [ap-northeast-2]: **엔터***
>
> *Default output format [json]: **엔터***

<br>

### 버킷 연동 테스트

boto3 SDK를 통해 내가 생성한 S3 버킷에 연동할 수 있는지 REPL을 통해 확인해보자.

<br>

> **$ python**
>
> `>>> import boto3
>
> `>>> s3 = boto3.resource('s3')
>
> `>>> for bucket in s3.buckets.all():
>
> `>>> 	print(bucket.name)

<br>

근데 에러난다. ;(

<br>

> *botocore.exceptions.ClientError: An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied*

<br>

### AWS CLI에 구성된 자격 증명 확인

AWS CLI에 구성된 

<br>

> **$ aws configure list**

```
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************HTVN shared-credentials-file
secret_key     ****************I4We shared-credentials-file
    region           ap-northeast-2      config-file    ~/.aws/config
```

<br>

> **$ aws sts get-caller-identity**

<br>

```json
{
    "UserId": "AIDAQU6BXDQ7HHG7KM5DU",
    "Account": "044966550590",
    "Arn": "arn:aws:iam::044966550590:user/django-board"
}
```

<br>

## AWS Lambda

Lambda는 AWS에서 제공하는 `Serverless`, `FaaS(Functions as a Service)` 서비스이다. 

<br>

### Serverless란?

Microservice

Stateless Functions

Cold Starts





