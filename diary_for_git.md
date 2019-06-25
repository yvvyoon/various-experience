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

현재 공통된 commit point가 물려있지 않기 때문에 나타나는 상황이 맞는 것 같다.

```
$ git pull web master --allow-unrelated-histories
```

빈 디렉토리에 위의 명령어를 실행하면 원격 Repo의 파일들을 전부 당겨오게 되는데 마찬가지로 이 또한 내가 로컬에 관리하던 파일들과 다르다.

결국, 나보다 먼저 노가다 씨름을 했었던 학원 동기에게 물어물어 방법을 찾아냈다.

원격 Repo의 파일명을 바꿔주고 최대한 로컬과 동일하게 싱크를 맞춘 다음 clone을 수행해볼 예정이다.

```
$ git clone https://github.com/yvvYoon/Web.git
```



결국!

clone으로 해결됐다.

변경사항이 적용되어야 하는 파일이 대략 7개정도 있었는데, 이상하게 1개 파일만 add하고 commit -m '메시지'를 실행했더니 나머지 6개 파일에 모두 공통으로 적용돼버렸다. 뭐지

이건 GitHub에서 수작업으로 고쳐주면 되니까 이번 이슈는 어딘가 모르게 찝찝하지만 일단 종료되었다.

