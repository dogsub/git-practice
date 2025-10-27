# GitHub Repository 생성 후 작업 준비까지

// Git 관리 시작
git init

// Git 레포 작업 전 미리 .gitignore 파일 작성
nano .gitignore

// .gitignore 파일 스테이징 및 커밋
git add .gitignore
git commit -m "Initialize project with .gitignore"

// 메인 브랜치명 설정
git branch -M main

// 원격 저장소 연결
git remote add origin ${새 레포 주소.git}

// 원격 저장소로 첫 푸시
git push -u origin main

---

# 기존 프로젝트 Develop하기 위한 GitHub Repository 생성 후 작업 준비까지

## 이때 이전의 git comiit 이력을 모두 지우고 싶다면 .git 폴더 삭제 후 다시 git init 해야 됨!!

// 기존 프로젝트 클론
git clone ${기존 레포 주소}
cd {프로젝트 폴더}

// 기존 원격 확인 및 제거
git remote -v       # 기존 origin 주소 확인
git remote remove origin

// 새 원격 저장소 연결
git remote add origin ${새 레포 주소.git}

// 메인 브랜치명 설정
git branch -M main

// 커밋 & 푸시
git add .
git commit -m "Initialize project"
git push -u origin main
