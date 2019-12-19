## MySQL 계정 관련

EC2에 프로젝트를 새로 적재하든, 새로운 개발 환경에서 MySQL을 구축하든 여러 상황에서 계정, 데이터베이스를 생성하면서 매번 구글링하는 게 귀찮아지는 경지에 이르렀다. 그렇다고 내 기억력이 좋은 편도 아니다.

그래서 정리해봤다. 내가 나중에 쓰려고.

난 5.8 버전으로 설치했기 때문에 미만의 버전에서는 통하지 않을 수 있다. 5.7 이하의 버전은 다른 방법이 있다고 한다.

이상하게 백지 상태에서 클린하게 설치했는데도 root 비번이 맞지 않는다는 아이러니한 상황이 발생했다. 그냥 엔터를 치든 root로 입력을 하든 별 수단을 다 해봐도 안 먹히길래 아래와 같은 방법으로 해결했다.

### root 계정 패스워드 변경

```
$ sudo mysql -u root
```

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY 'root';
mysql> FLUSH PRIVILEGES;
```

<br>

### 데이터베이스 생성

```
mysql> CREATE DATABASE djboard CHARACTER SET utf8 COLLATE utf8_bin;
```

<br>

### 계정 생성

```
mysql> CREATE USER 'yoon'@'localhost' IDENTIFIED BY 'yoon' PASSWORD EXPIRE NEVER;
mysql> GRANT ALL PRIVILEGES ON djboard.* TO 'yoon'@'localhost' IDENTIFIED BY 'yoon';

MySQL을 docker 컨테이너로 사용하려고 세팅하려니 위 grant문이 먹히지 않았다. 아래 커맨드로 하니 정상적으로 부여된다.

mysql> GRANT ALL PRIVILEGES TO djboard.* TO 'yoon'@'localhost' WITH GRANT OPTION;
```