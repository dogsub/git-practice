# Git의 push 거절과 병합 충돌 구분하기

## 1. non-fast-forward와 conflict는 다르다

- **non-fast-forward 거절**: 원격 브랜치의 기존 끝 커밋이 새로 push할 로컬 끝 커밋의 조상이 아니어서, 원격 이력이 사라질 수 있는 갱신을 거절하는 것이다.
- **병합 충돌(conflict)**: 변경을 통합할 때 Git이 결과를 자동으로 결정하지 못하는 것이다.

push가 거절되어도 fetch 후 merge나 rebase가 충돌 없이 끝날 수 있다. 같은 파일을 수정했다고 반드시 충돌하는 것도 아니다.

## 2. 예제: 양쪽에서 커밋을 만든 상황

로컬 main과 서버 main이 같은 커밋 O에서 시작했고 작업 트리가 깨끗하다고 가정한다.

```bash
git switch main
nano edition.txt
git add edition.txt
git commit -m "Update edition locally"
```

이후 다른 사용자가 GitHub 웹에서 edition.txt의 **같은 부분을 다른 내용으로** 수정해 main에 커밋한다. 웹 커밋은 서버에 바로 기록되므로 별도 로컬 push 단계는 없다.

```text
      L  (로컬 main)
     /
O---+
     \
      R  (서버 main)
```

로컬과 원격에는 서로 없는 커밋이 있다. 단순히 로컬이 뒤처진 상태가 아니라 **이력이 분기(diverged)된 상태**다. 아직 fetch하지 않았다면 로컬 `origin/main`은 O를 가리킬 수 있다.

```bash
git push origin main
```

일반 push는 `(fetch first)` 또는 `(non-fast-forward)` 등의 메시지와 함께 거절될 수 있다. 이 단계는 내용 충돌을 해결하는 단계가 아니다.

## 3. fetch 후 merge

```bash
git fetch origin
git log --oneline --graph --decorate --all
git merge origin/main
```

fetch로 로컬 `origin/main`이 R을 가리키게 되지만, 현재 main과 작업 트리에는 아직 통합되지 않는다.

일반적인 3-way merge는 **공통 조상 O, 현재 브랜치 L, 합칠 대상 R**을 비교한다. O에서 양쪽으로 생긴 변경을 자동 통합하려 시도하고, 같은 부분의 서로 다른 수정 등은 충돌로 남긴다.

```text
CONFLICT (content): Merge conflict in edition.txt
Automatic merge failed; fix conflicts and then commit the result.
```

대표적인 충돌 상황은 다음과 같다.

| 상황 | 자동 결정이 어려운 이유 |
| :--- | :--- |
| 같은 부분을 서로 다르게 수정 | 어느 내용을 남기거나 결합해야 하는지 알 수 없음 |
| 한쪽은 파일 삭제, 다른 쪽은 수정 | 삭제와 수정 유지 중 선택 필요 |
| 같은 경로에 서로 다른 파일을 추가 | 최종 파일 내용 결정 필요 |
| 같은 파일을 서로 다른 이름으로 변경 | 최종 경로 결정 필요 |
| 양쪽에서 같은 바이너리 파일을 수정 | 기본 텍스트 병합으로 내용을 합치기 어려움 |

## 4. 충돌 해결 및 병합 완료

```bash
git status
git diff --name-only --diff-filter=U
```

텍스트 충돌은 기본 설정에서 다음과 같이 표시될 수 있다.

```text
<<<<<<< HEAD
로컬 main의 내용
=======
origin/main의 내용
>>>>>>> origin/main
```

이 merge 예제에서 위쪽은 현재 main, 아래쪽은 origin/main이다. 설정에 따라 공통 조상 내용이 추가로 보일 수 있고, 삭제·이름 변경·바이너리 충돌은 이런 표식만으로 표현되지 않을 수도 있다.

1. 양쪽 변경 의도를 확인하고 최종 내용을 작성한다. 한쪽 전체를 고르는 것만이 해결 방법은 아니다.
2. 텍스트 충돌 표식을 제거하고 필요한 테스트나 실행으로 결과를 확인한다.
3. 해결한 파일을 스테이징하고 병합 커밋을 만든다.

```bash
git add edition.txt
# 삭제로 해결한 파일은 git rm <파일> 사용
git status
git diff --cached --check
git diff --cached
git commit -m "Resolve merge conflict in edition"
git push origin main
```

`git add`는 해결 결과를 인덱스에 등록한다. 내용이 논리적으로 올바른지까지 검증하지는 않는다. `--check` 역시 공백 오류나 충돌 표식 등을 찾는 보조 수단이며 기능 검증을 대신하지 않는다.

```text
      L---M  (main, push 성공 후 서버 main)
     /   /
O---+   /
     \ /
      R
```

M은 L과 R을 부모로 갖는 merge 커밋이다. push 전에 다른 사람이 원격 main을 갱신하면 다시 거절될 수 있으므로 새 변경을 가져와 통합한다. 성공한 push는 해당 브랜치가 그 시점에 같은 커밋을 가리킨다는 뜻이며, 모든 브랜치와 미커밋 파일까지 동기화되었다는 뜻은 아니다.

병합을 중단하려면 진행 중인 상태에서 `git merge --abort`를 실행한다. 병합 전 미커밋 변경이 섞여 있으면 완전한 복원이 어려울 수 있으므로 먼저 커밋하거나 stash하는 것이 좋다.

## 5. 로컬에 별도 커밋이 없으면 fast-forward

서버에만 새 커밋이 있고 로컬 main이 그 조상인 경우에는 포인터를 앞으로 옮길 수 있다. 작업 트리가 깨끗한 상태에서 실행한다.

```bash
git switch main
git fetch origin
git merge --ff-only origin/main
```

다음 명령으로 같은 목적을 수행할 수도 있다.

```bash
git pull --ff-only origin main
```

fast-forward는 새로운 merge 커밋을 만들지 않는다. `pull`은 fetch 후 옵션·설정에 따라 merge 또는 rebase 등으로 통합하므로, 항상 `fetch + merge`라고 단정하지 않는다. `git pull origin`처럼 브랜치를 생략하면 설정에 의존한다. 또한 pull은 **현재 브랜치**에 통합하므로 main을 최신화하려면 먼저 main으로 이동한다.

## 6. rebase 중 충돌이라면

rebase도 커밋 재적용 도중 충돌할 수 있다. 해결 후 `git add <파일>`과 `git rebase --continue`로 진행하고, 중단하려면 `git rebase --abort`를 사용한다. 자세한 절차와 공유 이력 주의점은 [Git-Rebase.md](Git-Rebase.md)를 참고한다.

자동 병합에 성공했어도 한쪽의 함수 이름 변경과 다른 쪽의 새 호출 코드가 맞지 않는 등 기능상의 문제가 남을 수 있다. 충돌 표식이 없다는 사실과 프로그램이 올바르게 동작한다는 사실은 별개다.

## 면접 답변 예시

> push 거절은 이력 관계에 대한 검사이고, 병합 충돌은 내용을 통합할 때 생기는 문제입니다. 원격 커밋이 로컬 이력에 포함되지 않으면 일반 push는 원격 이력 보호를 위해 거절될 수 있습니다. 먼저 fetch로 원격 상태를 확인하고 merge나 rebase로 통합합니다. 그 과정에서 공통 조상 대비 양쪽의 변경을 자동으로 조합할 수 없으면 충돌을 해결하고 테스트한 다음 작업을 완료합니다.

## 공식 참고 자료

- [git merge](https://git-scm.com/docs/git-merge)
- [git push](https://git-scm.com/docs/git-push)
- [git pull](https://git-scm.com/docs/git-pull)
