## GitHub 초보 탈출기

그 동안 T4IR라는 무슨 의미인지 1도 파악할 수 없는 나만 아는 이름의 Repo를 사용하다가 공부하는 영역별 Repo들로 재구성하는 중이다.

유일했던 T4IR Repo에 죄다 몰아서 올리기만 하다가 처음으로 2개로 분리해서 각 Repo에 올리는 법을 찾고 있었다.

Git의 기능을 뭘 모르는지 몰랐기 때문에 구글링을 헤매다가 현업 개발자 친구 도움으로 원격저장소(Remote)를 이용하기로 했다.

```
$ git remote add algo https://github.com/yvvYoon/Algorithm.git
$ git add 01_List.md
$ git commit -m 'Functions and comparisons of List'
$ git push algo master
```

> To https://github.com/yvvYoon/Algorithm.git
>
> ! [rejected]        master -> master (fetch first)
>
> error: 레퍼런스를 'https://github.com/yvvYoon/Algorithm.git'에 푸시하는데 실패했습니다
>
> 힌트: 리모트에 로컬에 없는 사항이 들어 있으므로 업데이트가
>
> 힌트: 거부되었습니다. 이 상황은 보통 또 다른 저장소에서 같은
>
> 힌트: 저장소로 푸시할 때 발생합니다.  푸시하기 전에
>
> 힌트: ('git pull ...' 등 명령으로) 리모트 변경 사항을 먼저
>
> 힌트: 포함해야 합니다.
>
> 힌트: 자세한 정보는 'git push --help'의 "Note about fast-forwards' 부분을
>
> 힌트: 참고하십시오.	

위기다.

하라는대로 했는데.

README.md 내용을 수정한 적이 있는데 그 내용이 로컬에 반영되지 않았고, 싱크가 맞지 않아 발생한 에러인 것 같다.

remote = local로 맞춰주기 위해 git pull을 사용한다.

```
$ git pull algo master
```

>  ! [rejected]        master -> master (non-fast-forward)error: 레퍼런스를 
>
>  'https://github.com/yvvYoon/Algorithm.git'에 푸시하는데 실패했습니다
>
>  힌트: 현재 브랜치의 끝이 리모트 브랜치보다 뒤에 있으므로 업데이트가
>
>  힌트: 거부되었습니다. 푸시하기 전에 ('git pull ...' 등 명령으로) 리모트
>
>  힌트: 변경 사항을 포함하십시오.
>
>  힌트: 자세한 정보는 'git push --help'의 "Note about fast-forwards' 부분을
>
>  힌트: 참고하십시오.



위 문제는 아직 해결하지 못했고 어찌저찌 하다보니 다른 에러가 발생하고 있다.

기존의 T4IR Repo의 이름을 Web으로 수정했고, 내 로컬의 Web 디렉토리에 있는 파일들의 이름을 일부 변경했었다.

그래서 현재 원격 Remote와 로컬의 싱크가 맞지 않아 pull을 시도했을 때 아래와 같은 에러가 발생하는 것 같다.

```
$ git pull web master
```

> https://github.com/yvvYoon/Web URL에서
>
> \* branch            master     -> FETCH_HEAD
>
> fatal: 관계 없는 커밋 내역의 병합을 거부합니다	





