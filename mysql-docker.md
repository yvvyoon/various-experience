## MySQL을 Docker 컨테이너로!

이전에 매번 MySQL 계정 및 데이터베이스 설정하는 게 귀찮아 Github에 올렸던 적이 있다. 이제 데이터 쌓는 것까지 너무 귀찮아서 데이터까지 통째로 포함하여 Docker 컨테이너를 pull/push하려고 한다.

일단 내 Docker Hub에 이미지 올렸던 경험은 있는데 데이터까지 말아서 올릴 수 있는지는 아직 확인해보지 않았다.

<br>

```
$ docker pull mysql
$ docker run --name mysql-db -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql
```

<br>

`e` 옵션은 `Environment Variables`의 약어로서 총 4개의 변수를 포함한다.

<br>

> *MYSQL_ROOT_PASSWORD (필수)*
>
> *MYSQL_DATABASE: 데이터베이스 생성 시 사용*
>
> *MYSQL_USER, MYSQL_PASSWORD: 사용자를 생성하고 생성된 데이터베이스의 권한을 얻음*
>
> *MYSQL_ONETIME_PASSWORD: 1회성 패스워드 생성*

<br>

```
$ docker exec -it mysql-db bash
```

<br>

CLI로 들어왔으면 MySQL에 접속해서 데이터베이스와 사용자를 생성해준다. 이제 내가 구축한 서버에서 이 MySQL 컨테이너로 접속하려면 이 컨테이너의 가상 IP를 알아내야 한다. 아래 커맨드는 `특정 컨테이너`의 네트워크 정보를 조회하는 것이다.

`json.` 다음에 띄어쓰기 꼭 넣어! 객체 참조가 아니라 인자로 받는 것이다.

```
$ docker inspect mysql-db -f "{{json. NetworkSettings.Networks}}"
```

```json
{"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"NetworkID":"8b21c5994c4eb65a3ec24af1d68c1c171553049741d1262e4a23b1de630782a1","EndpointID":"adb3c07e49cd454c04de20de69a964af3f52350513ea544d4216894cce24ea2a","Gateway":"172.17.0.1","IPAddress":"172.17.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02","DriverOpts":null}}
```

예쁘지 않다. 눈 아픔.

<br>

```
$ docker inspect mysql-db -f "{{json. NetworkSettings.Networks}}" | json_pp
```

```json
{
   "bridge" : {
      "Links" : null,
      "IPAddress" : "172.17.0.2",
      "Gateway" : "172.17.0.1",
      "DriverOpts" : null,
      "EndpointID" : "adb3c07e49cd454c04de20de69a964af3f52350513ea544d4216894cce24ea2a",
      "IPAMConfig" : null,
      "Aliases" : null,
      "MacAddress" : "02:42:ac:11:00:02",
      "IPv6Gateway" : "",
      "IPPrefixLen" : 16,
      "GlobalIPv6PrefixLen" : 0,
      "GlobalIPv6Address" : "",
      "NetworkID" : "8b21c5994c4eb65a3ec24af1d68c1c171553049741d1262e4a23b1de630782a1"
   }
}
```

<br>

위의 IP로 MySQL 컨테이너에 접속을 테스트해봤는데 실패했다. Flask에서 migrate도 먹히질 않는다. 왜지. 방화벽인가. 인바운드 때문인가.

