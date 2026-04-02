# Git rebase 정리: 내 디렉터리 작업은 유지하고, 다른 사람 작업은 최신 main 반영하기

## 상황 예시

- **3월 20일**: 내가 `dongseob/` 디렉터리에서 작업함 (`PR` 머지 전)
- **3월 22일**: A가 `apple/` 디렉터리에서 작업 후 `PR` 머지
- **3월 23일**: B가 `bear/` 디렉터리에서 작업 후 `PR` 머지

---

## 현재 상태

### 원격 `main`
- `apple/`는 최신 상태
- `bear/`도 최신 상태
- `dongseob/`는 내 추가 작업이 아직 머지되지 않았으므로 `main`에는 없음

### 내 로컬 작업 브랜치
- `dongseob/`에는 내 변경 사항이 있음
- `apple/`, `bear/`는 예전 상태라서 현재 `main`보다 뒤처져 있음

---

## 내가 원하는 결과
### 단, 이때의 전제는 각자 서로 다른 디렉터리에서만 작업을 했다는 가정 하에 둔다.

- `dongseob/`는 내 작업을 그대로 유지하고 싶다
- `apple/`, `bear/`는 최신 `main` 상태를 반영하고 싶다

이럴 때는 **내 작업 브랜치를 최신 `origin/main` 기준으로 rebase** 하면 된다.

---

## 실행 명령어

내 작업 브랜치(예: `dongseob`)에서 아래 명령어를 실행한다.

```bash
git fetch origin
git rebase origin/main


⸻

rebase 전후 개념

rebase 전

main:    --- A(apple) --- B(bear)
branch:  --- X(dongseob)

	•	main에는 A와 B의 작업이 반영되어 있음
	•	내 브랜치에는 dongseob/ 관련 작업 X가 있음
	•	아직 내 작업은 main에 합쳐지지 않음

rebase 후

main:    --- A(apple) --- B(bear)
branch:  ----------------------- X'(dongseob)

	•	apple/, bear/는 최신 main 기준으로 맞춰짐
	•	내 dongseob/ 작업은 그 최신 상태 위에 다시 얹힘

즉, 결과적으로 다음 상태가 된다.
	•	dongseob/ → 내 작업 유지
	•	apple/, bear/ → 최신 main 반영

⸻

중요한 전제

이 방법은 내 작업이 아직 main에 머지되지 않은 개인 작업 브랜치일 때 가장 자연스럽다.

즉, 아래 조건일 때 사용한다.
	•	내 작업은 아직 작업 브랜치에만 있다
	•	다른 사람 작업은 이미 main에 머지되었다
	•	나는 내 작업을 유지하면서 main의 최신 상태를 반영하고 싶다

⸻

이미 내 작업도 main에 머지된 경우

만약 내 작업도 이미 main에 머지된 상태라면, 굳이 rebase할 필요는 없다.

그 경우에는 그냥 main만 최신화하면 된다.

```bash
git switch main
git pull origin main
