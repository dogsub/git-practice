# Git rebase: 내 커밋을 최신 main 위에 다시 적용하기

## 상황과 목적

- 3월 20일: 내 작업 브랜치 `dongseob`에서 `dongseob/`를 수정해 커밋 X를 만든다.
- 3월 22일: A의 `apple/` 작업이 main에 병합된다.
- 3월 23일: B의 `bear/` 작업이 main에 병합된다.

내 작업은 유지하면서 최신 main의 변경도 반영하려면 작업 브랜치를 `origin/main` 위로 rebase할 수 있다. rebase는 디렉터리를 선택해서 동기화하는 기능이 아니라 **브랜치의 커밋을 새로운 기반 위에 다시 적용하는 작업**이다.

서로 독립적인 파일을 수정했다면 대체로 자동 적용되지만, 디렉터리가 다르다는 이유만으로 기능 호환성까지 보장되지는 않는다. 같은 파일을 수정했어도 변경 부분이 겹치지 않으면 충돌 없이 적용될 수 있다.

## 실행 전 준비

```bash
git switch dongseob
git status
```

미커밋 변경이 있으면 먼저 커밋하거나 `git stash push -u -m "before rebase"`로 임시 보관한다. `-u`는 미추적 파일도 포함하지만 무시된 파일은 포함하지 않는다. 아래 절차는 작업 트리가 깨끗한 상태를 전제로 한다.

```bash
git fetch origin
git rebase origin/main
```

fetch는 원격 추적 참조를 갱신하고, rebase는 현재 작업 브랜치의 이력을 변경한다. 로컬 main이나 서버 main을 직접 갱신하는 작업은 아니다.

## 커밋 그래프의 변화

아래 A·B는 main에 추가된 커밋을 단순화한 표현이며, O는 공통 조상이다.

```text
rebase 전
      A---B  (origin/main)
     /
O---+
     \
      X      (dongseob, HEAD)

rebase 후
O---A---B    (origin/main)
         \
          X' (dongseob, HEAD)
```

X의 변경을 B 위에 적용해 X'를 만든다. 부모 커밋이 달라지므로 커밋 ID도 달라진다. 이 예에서는 main의 apple/·bear/ 변경과 내 dongseob/ 변경이 함께 반영된다. 이미 기반에 반영된 동등한 변경 등은 rebase 과정에서 생략될 수 있다.

## 충돌이 발생하면

```bash
git status
# 충돌한 파일을 열고 양쪽 의도를 반영하여 편집
# 충돌 표식도 제거
git add <해결한파일>
git rebase --continue
```

삭제가 올바른 해결이라면 해당 파일을 `git rm`으로 제거한다. 적용할 커밋이 여러 개이면 중간에 충돌이 반복될 수 있다. 일반 merge와 달리 여기서는 별도의 merge 커밋을 만들지 않고 `rebase --continue`로 이어 간다.

- 취소하고 시작 전으로 돌아가기: `git rebase --abort`
- 현재 적용 중인 커밋을 생략하기: `git rebase --skip` — 그 커밋의 변경을 제외해도 되는 경우에만 사용한다.
- rebase 중 `ours`는 기반과 지금까지 재적용된 쪽, `theirs`는 현재 재적용 중인 내 커밋 쪽이므로 이름만 보고 선택하지 않는다.

완료 후 로그와 실제 동작을 확인한다. 미리 stash했다면 `git stash pop`으로 복원하며, 이 단계에서도 충돌이 발생할 수 있다. 충돌 시 stash 항목이 남을 수 있으므로 해결 후 `git stash list`로 확인한다.

## 원격 작업 브랜치 갱신

처음 올리는 브랜치는 일반 push를 사용한다.

```bash
git push -u origin dongseob
```

이미 push한 커밋을 rebase하면 이력이 달라져 일반 push가 거절될 수 있다. 다른 사람이 기반으로 사용하지 않는 개인 작업 브랜치이며 팀 규칙이 허용할 때만 다음을 사용한다.

```bash
git push --force-with-lease origin dongseob
```

`--force-with-lease`는 원격 참조가 기대한 상태일 때만 갱신하도록 보호한다. 기본형은 로컬 원격 추적 참조를 기대값으로 사용하므로 백그라운드 fetch가 이를 갱신하면 보호 의도가 약해질 수 있다. 공유 main의 이력을 임의로 재작성하는 용도로 사용하지 않는다.

## merge와의 차이

| 방식 | 이력에 미치는 영향 | 고려할 상황 |
| :--- | :--- | :--- |
| `git merge origin/main` | 기존 커밋을 유지. 이력이 갈라졌다면 일반적으로 merge 커밋 생성 | 여러 사람이 사용하는 브랜치 |
| `git rebase origin/main` | 내 커밋을 새 기반에 재적용해 이력 재작성 | 개인 작업 브랜치의 이력 정리 |

merge도 fast-forward 가능한 경우 새 커밋 없이 포인터만 이동할 수 있다. rebase는 충돌을 예방하는 명령이 아니며, 어느 방식이든 통합 결과를 검증해야 한다.

## 내 작업도 이미 main에 병합되었다면

후속 작업이 없는 경우 main을 최신화하고 새 작업 브랜치를 만든다.

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/next-work
```

로컬 main에 별도 커밋이 있어 분기되었다면 `--ff-only`는 실패한다. 이력을 확인한 뒤 통합 방식을 결정한다. 기존 작업 브랜치에 미병합 후속 커밋이 있거나 squash merge를 사용했다면 단순히 모든 작업이 끝났다고 가정하지 않는다.

## 면접 답변 예시

> merge는 기존 커밋을 유지하면서 이력을 통합하고, rebase는 커밋을 다른 기반 위에 재적용하여 이력을 재작성합니다. 그래서 이미 여러 사람이 사용 중인 커밋의 rebase는 협업자의 이력과 불일치를 일으킬 수 있습니다. 개인 브랜치 정리에는 rebase를 사용할 수 있지만 공유 브랜치에서는 팀 정책을 따르며, 충돌이 발생하면 해결 후 add와 rebase --continue로 진행합니다.

## 공식 참고 자료

- [git rebase](https://git-scm.com/docs/git-rebase)
- [git push: force-with-lease](https://git-scm.com/docs/git-push)
