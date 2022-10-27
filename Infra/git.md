# 일반
## 용어
HEAD
- 현재 브랜치 마지막 커밋 스냅샷(파일 묶음)을 가리키는 포인터.
- 위치 : `.git/HEAD`
Index
- 바로 다음 커밋의 스냅샷(파일 묶음)을 가리키는 포인터. Staging Area. `git commit` 실행 시 git이 새 커밋 대상이 있는 공간.
- 위치 : `.git/index`
```bash
$ git ls-files -s
100644 a906cb2a4a904a152e80877d4088654daad0c859 0	README
100644 8f94139338f9404f26296befa88755fc2598c289 0	Rakefile
100644 47c6340d6459e05787f644c2447d2595f5d3a54b 0	lib/simplegit.rb
```
워킹 디렉터리 : 샌드박스. 스테이징하지않은 모든 코드
![[Pasted image 20221027095321.png]]
## 일반적인 커밋 흐름
1. `git init`
![[Pasted image 20221027095412.png]]
2. `git add`
![[Pasted image 20221027095448.png]]
3. `git commit`
![[Pasted image 20221027095529.png]]
4. 파일 변경
![[Pasted image 20221027095556.png]]
5. 다시 `git add`
![[Pasted image 20221027095619.png]]
6. 다시 `git commit`
![[Pasted image 20221027095718.png]]


## git reset
[git-scm : Git 도구 Reset 명확히 알고가기](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Reset-%EB%AA%85%ED%99%95%ED%9E%88-%EC%95%8C%EA%B3%A0-%EA%B0%80%EA%B8%B0)
역할 : 현재 브랜치의 직전 커밋 히스토리 상태로 돌아간다
옵션
1. `git reset --soft` : head를 직전커밋으로 변경
![[Pasted image 20221027094401.png]]
2. `git reset --mixed` : head를 직전커밋으로 + Staging area(=index)를 직전 상태로
![[Pasted image 20221027094454.png]]
3. `git reset --hard` : head + Staging area(index) + Working Directory 까지 직전 커밋으로
![[Pasted image 20221027094557.png]]
# Errors
- Error: bed index - Fatal: index file corrupt
1. 기존 git 디렉터리 백업
2. `.git/index` 삭제 : 리눅스면 `rm -f .git/index`, 윈도우면 `del .git\index`
3. `git reset` : 직전 커밋 상태로 돌리기
