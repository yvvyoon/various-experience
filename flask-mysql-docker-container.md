## Flask - MySQL 연동

로컬 환경에서 연동한다는 것이 아니라 각 컨테이너를 띄운 후 컨테이너끼리 통신하여 연동하는 방법을 적용할 것이다. 쉽게 연동하기 위해 `docker-compose.yml` 파일을 사용할 것이고, 프로젝트 루트 디렉토리에 생성한다. 학원에서 많이 만져봤던 기억이 난다 새록새록.

```yaml
version: "2"
services:
  app:
    build: ./app
    links:
      - mysql-db
    ports:
      - "5000:5000"
  db:
    image: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ./db:/docker-entrypoint-initdb.d/:ro

```

<br>

> `build`: Dockerfile 파일을 포함하고 있는 디렉토리
>
> `links`: Flask 컨테이너를 띄울 때 설정한 Link의 이름을 명시. IP가 아니라 컨테이너의 이름으로 연결할 수 있을 뿐만 아니라 컨테이너의 시작 순서를 결정하는 종속성을 표현할 수 있음
>
> `ports`: \<host>:\<container>
>
> `image`: 

<br>