# GitHub 원격 저장소(`origin/main`)와 로컬(`main`) 간의 충돌(conflict) 발생 및 해결

## 1️⃣ 초기 상태

* 로컬과 원격 모두 `main` 브랜치 존재
* 로컬에서 기존에 존재하던 파일(`edition.txt`)을 수정 후 커밋

```
nano edition.txt
git add .
git commit -m "hotfix my solution"
```

→ 로컬이 원격 저장소(Remote Repository)보다 한 단계 앞선 상태

---

## 2️⃣ 다른 사용자가 GitHub 웹에서 파일(`edition.txt`) 직접 수정

→ 원격 저장소(Remote Repository)는 로컬의 커밋이 아직 적용 X

---

## 3️⃣ 로컬에서의 푸시 실패 (non-fast-forward)

```
git push origin main
```

* 오류:

```
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/dogsub/git-practice.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
  
* 의미:
  → **GitHub 웹에서 직접 수정한 커밋이 원격에 존재**
  → 로컬 `main`이 원격 `main`보다 **뒤처진 상태**
  → 즉, 서로 다른 커밋을 가진 “분기(diverged)” 상태 발생

---

## 4️⃣ 원격 변경 가져오기 (fetch)

```
git fetch origin
```

* 원격의 새 커밋(`82e42de`)을 받아서 **`origin/main` 포인터 갱신**
* 아직 로컬 `main`에는 병합되지 않음 (즉, 작업 트리는 그대로)

---

## 5️⃣ 병합 시도 및 충돌 발생

```
git merge origin/main
```

* 결과:

  ```
  CONFLICT (content): Merge conflict in edition.txt
  Automatic merge failed; fix conflicts and then commit the result.
  ```
* 의미:
  → **edition.txt 파일을 로컬과 원격에서 모두 수정했음**
  → Git이 자동 병합 실패 → 수동 충돌 해결 필요

---

## 6️⃣ 충돌 해결 및 커밋


1. `git status`로 현재 상태를 파악
2. `nano edition.txt` 로 파일 열기
   → 충돌 표식(`<<<<<<<`, `=======`, `>>>>>>>`) 제거 후 수동 편집
3. 수정 완료 후 단계별로 수행:

   ```
   git add edition.txt
   git status    # 충돌 해결되었는지 확인
   git commit -m "resolved conflict"
   ```

* 이제 병합 완료 ✅
* `main` 브랜치에 병합 커밋 생성됨

4. 원격 저장소에 푸시하기

   ```
   git push origin main
   ```

  * 로그:

  ```
  To https://github.com/dogsub/git-practice.git
   82e42de..3ba95a8  main -> main
  ```

  * 의미:
  → 로컬과 원격의 이력이 병합되어 정리 완료
  → **GitHub와 완전히 동기화된 상태**
   
---

# GitHub 원격 저장소(`origin/main`)의 작업 상황 로컬로 가져오기

## 1️⃣ GitHub 웹(=원격 저장소)에서 파일(`edition.txt`) 편집 및 커밋 그리고 푸시

* 웹에서 `edition.txt`를 또 수정했다고 가정

---

## 2️⃣ 로컬 저장소 동기화

* `git fetch origin` → 새 커밋(`~~~~~~`) 확인

* `git merge origin/main` → Fast-forward 병합 발생

  ```
  Updating 3ba95a8..6fb0088
  Fast-forward
  ```

  → 충돌 없이 자동 업데이트 완료

* 위 2개의 명령어와 `git pull origin`은 거의 동일하지만 위 2개의 명령어로 진행하는 것을 권장

✅ **최종 상태:**

* 로컬과 원격이 동일 (`origin/main` = `main`)
* 모든 충돌 해결 및 이력 정리 완료
