# GitHub 저장소 작업 시작하기

아래 URL의 `YOUR_NAME`, `YOUR_REPOSITORY`는 실제 값으로 바꾼다. `${새 레포 주소.git}`는 올바른 셸 자리표시자가 아니므로 사용하지 않는다.

## 1. 새로운 프로젝트를 새 GitHub 저장소에 올리기

```bash
git init -b main

# 필요한 경우 이 저장소의 커밋 작성자 설정
git config user.name "Your Name"
git config user.email "you@example.com"

# 프로젝트에 맞춰 무시 규칙 작성
nano .gitignore

git add .gitignore
git commit -m "Initialize project with .gitignore"

git remote add origin https://github.com/YOUR_NAME/YOUR_REPOSITORY.git
git push -u origin main
```


## 2. 기존 프로젝트를 새 저장소[레포]로 옮기기 with 커밋 유지

```bash
git clone https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git my-project
cd my-project

# 현재 브랜치 확인 후 만약 메인 브랜치가 아니라면 main으로
git branch
git switch main

# 원본 연결 유지
git remote rename origin upstream

# 새 저장소 연결
git remote add origin https://github.com/YOUR_NAME/YOUR_NEW_REPOSITORY.git

# 원본 커밋 이력 유지
git push -u origin main
```


## 3. 기존 프로젝트를 새 저장소[레포]로 옮기기 with 원본 연결 해제 & 커밋 유지 X

```bash
git clone https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git original-project

# .git을 제외하고 새로운 my-project 폴더로 복사
rsync -av --exclude='.git' original-project/ my-project/
mkdir my-project
cd my-project

# 이후 "1. 새로운 프로젝트를 새 GitHub 저장소에 올리기"와 동일하게 진행
```