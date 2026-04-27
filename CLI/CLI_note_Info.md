whoami
id
cat /etc/passwd
uname -a
cat /etc/*release
rpm -qa
cat /proc/cpuinfo



## 1. `whoami`

현재 로그인한 사용자의 이름을 출력하는 명령어 - 현재 내가 어떤 사용자 계정으로 명령어를 실행하고 있는지 확인

```bash
whoami
```

---

## 2. `id`

현재 사용자 또는 특정 사용자의 UID, GID, 소속 그룹 정보를 확인하는 명령어

```bash
id
```

예시 결과:

```bash
uid=1000(dongseob) gid=1000(dongseob) groups=1000(dongseob),10(wheel)
```

각 항목의 의미는 다음과 같다.

| 항목 | 의미 |
|---|---|
| `uid` | User ID, 사용자 식별 번호 |
| `gid` | Group ID, 기본 그룹 식별 번호 |
| `groups` | 사용자가 속한 전체 그룹 목록 |

예시를 해석하면 다음과 같다.

```text
uid=1000(dogsub)
```

현재 사용자의 UID는 `1000`이고, 사용자 이름은 `dogsub`이다.

```text
gid=1000(dogsub)
```

현재 사용자의 기본 그룹 GID는 `1000`이고, 기본 그룹 이름도 `dogsub`이다.

```text
groups=1000(dogsub),10(wheel)
```

현재 사용자는 `dogsub` 그룹과 `wheel` 그룹에 속해 있다.

RHEL/Rocky Linux 계열에서 `wheel` 그룹은 보통 `sudo` 권한과 관련이 있다.

특정 사용자의 정보를 확인할 수도 있다.

```bash
id root
```

예시 결과:

```bash
uid=0(root) gid=0(root) groups=0(root)
```

### 사용 목적

```text
현재 사용자의 UID/GID 확인
사용자가 어떤 그룹에 속해 있는지 확인
sudo 권한 그룹에 포함되어 있는지 확인
파일 권한 문제 분석
```

---

## 3. `cat /etc/passwd`

시스템에 등록된 사용자 계정 정보를 확인하는 명령어

```bash
cat /etc/passwd
```

`/etc/passwd` 파일은 Linux 시스템의 사용자 계정 정보를 저장하는 파일이다.

예시:

```text
root:x:0:0:root:/root:/bin/bash
dongseob:x:1000:1000:dogsub:/home/dongseob:/bin/bash
```

각 필드는 `:` 기호로 구분된다.

```text
사용자명:비밀번호:UID:GID:설명:홈 디렉터리:로그인 셸
```

예를 들어 아래 내용을 보면:

```text
dogsub:x:1000:1000:dogsub:/home/dogsub:/bin/bash
```

각 항목의 의미는 다음과 같다.

| 필드 | 예시 | 의미 |
|---|---|---|
| 사용자명 | `dogsub` | 로그인 계정 이름 |
| 비밀번호 | `x` | 실제 비밀번호는 `/etc/shadow`에 저장됨 |
| UID | `1000` | 사용자 ID |
| GID | `1000` | 기본 그룹 ID |
| 설명 | `dogsub` | 사용자 설명 또는 이름 |
| 홈 디렉터리 | `/home/dogsub` | 사용자 홈 디렉터리 |
| 로그인 셸 | `/bin/bash` | 로그인 시 사용하는 셸 |

### `/etc/passwd`에서 중요한 점

과거에는 비밀번호 해시가 `/etc/passwd`에 저장되었지만, 현재 Linux에서는 보안상 실제 비밀번호 해시는 `/etc/shadow`에 저장된다.

그래서 `/etc/passwd`의 두 번째 필드가 보통 `x`로 표시된다.

```text
root:x:0:0:root:/root:/bin/bash
```

여기서 `x`는 “비밀번호 정보는 `/etc/shadow`에 따로 있다”는 의미이다.

### 특정 사용자만 확인하기

```bash
grep dogsub /etc/passwd
```

예시:

```bash
grep root /etc/passwd
```

### 사용 목적

```text
시스템에 어떤 사용자 계정이 있는지 확인
특정 사용자의 UID/GID 확인
사용자의 홈 디렉터리 확인
사용자의 로그인 셸 확인
```

---

## 4. `uname -a`

커널 및 시스템 기본 정보를 출력하는 명령어

```bash
uname -a
```

예시 결과:

```bash
Linux server.server.local 5.14.0-570.12.1.el9_6.x86_64 #1 SMP PREEMPT_DYNAMIC Tue May 13 09:35:36 EDT 2025 x86_64 x86_64 x86_64 GNU/Linux
```

각 항목은 대략 다음과 같은 의미를 가진다.

| 항목 | 의미 |
|---|---|
| `Linux` | 커널 이름 |
| `server.server.local` | 호스트 이름 |
| `5.14.0-570.12.1.el9_6.x86_64` | 커널 버전 |
| `#1 SMP ...` | 커널 빌드 정보 |
| `x86_64` | 시스템 아키텍처 |
| `GNU/Linux` | 운영체제 유형 |

### 자주 사용하는 옵션

```bash
uname -s
```

커널 이름만 출력한다.

```bash
Linux
```

```bash
uname -r
```

커널 릴리즈 버전만 출력한다.

```bash
5.14.0-570.12.1.el9_6.x86_64
```

```bash
uname -m
```

시스템 아키텍처를 출력한다.

```bash
x86_64
```


### 사용 목적

```text
현재 커널 버전 확인
시스템 아키텍처 확인
OS가 64비트인지 확인
커널 업데이트 여부 확인
드라이버나 패키지 호환성 확인
```

---

## 5. `cat /etc/*release`

Linux 배포판 정보를 확인하는 명령어이다.

```bash
cat /etc/*release
```

`/etc` 아래의 `release`로 끝나는 파일들을 한 번에 출력한다.

대표적으로 다음 파일들이 포함될 수 있다.

```text
/etc/os-release
/etc/redhat-release
/etc/system-release
```

RHEL/Rocky Linux 계열 예시:

```text
Rocky Linux release 9.6 (Blue Onyx)
NAME="Rocky Linux"
VERSION="9.6 (Blue Onyx)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="9.6"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Rocky Linux 9.6 (Blue Onyx)"
```

### 주요 항목 의미

| 항목 | 의미 |
|---|---|
| `NAME` | 배포판 이름 |
| `VERSION` | 배포판 버전 |
| `ID` | 배포판 식별자 |
| `ID_LIKE` | 계열 정보 |
| `VERSION_ID` | 버전 번호 |
| `PRETTY_NAME` | 사람이 읽기 쉬운 배포판 이름 |

예를 들어:

```text
ID_LIKE="rhel centos fedora"
```

이 값은 해당 배포판이 RHEL, CentOS, Fedora 계열과 관련이 있음을 의미한다.

### 더 정확히 하나만 보고 싶다면

```bash
cat /etc/os-release
```

또는 RHEL 계열에서는:

```bash
cat /etc/redhat-release
```

예시:

```text
Rocky Linux release 9.6 (Blue Onyx)
```

### 사용 목적

```text
현재 Linux 배포판 확인
RHEL 계열인지 Debian 계열인지 확인
OS 버전 확인
패키지 설치 방식 확인
문서나 명령어 적용 가능 여부 판단
```

---

## 6. `rpm -qa`

RPM 기반 Linux에서 설치된 패키지 목록을 확인하는 명령어이다.

```bash
rpm -qa
```

`rpm`은 Red Hat 계열 Linux에서 사용하는 패키지 관리 도구이다.  
RHEL, Rocky Linux, AlmaLinux, CentOS, Fedora 계열에서 사용된다.

`-q`는 query, 즉 조회를 의미한다.  
`-a`는 all, 즉 전체 패키지를 의미한다.

따라서:

```bash
rpm -qa
```

는 “현재 시스템에 설치된 모든 RPM 패키지를 조회하라”는 뜻이다.

예시 결과:

```text
bash-5.1.8-9.el9.x86_64
coreutils-8.32-39.el9.x86_64
systemd-252-51.el9.x86_64
openssh-server-8.7p1-45.el9.x86_64
vim-enhanced-8.2.2637-22.el9.x86_64
```

### 특정 패키지 설치 여부 확인

패키지가 너무 많기 때문에 보통 `grep`과 함께 사용한다.

```bash
rpm -qa | grep ssh
```

예시 결과:

```text
openssh-8.7p1-45.el9.x86_64
openssh-server-8.7p1-45.el9.x86_64
openssh-clients-8.7p1-45.el9.x86_64
```

`httpd` 설치 여부 확인:

```bash
rpm -qa | grep httpd
```

`vim` 설치 여부 확인:

```bash
rpm -qa | grep vim
```

### 패키지 개수 확인

```bash
rpm -qa | wc -l
```

### 패키지 상세 정보 확인

```bash
rpm -qi 패키지명
```

예시:

```bash
rpm -qi openssh-server
```

### 특정 파일이 어느 패키지에 속하는지 확인

```bash
rpm -qf 파일경로
```

예시:

```bash
rpm -qf /usr/bin/ls
```

예시 결과:

```text
coreutils-8.32-39.el9.x86_64
```

즉 `/usr/bin/ls` 파일은 `coreutils` 패키지에 포함되어 있다는 의미이다.

### 사용 목적

```text
설치된 전체 패키지 확인
특정 프로그램 설치 여부 확인
패키지 버전 확인
파일이 어느 패키지에서 제공되었는지 확인
장애 분석 시 관련 패키지 설치 상태 확인
```

---

# 명령어 요약 표

| 명령어 | 용도 |
|---|---|
| `whoami` | 현재 로그인한 사용자 이름 확인 |
| `id` | UID, GID, 소속 그룹 확인 |
| `cat /etc/passwd` | 시스템 사용자 계정 정보 확인 |
| `uname -a` | 커널 및 시스템 기본 정보 확인 |
| `cat /etc/*release` | Linux 배포판 및 버전 정보 확인 |
| `rpm -qa` | 설치된 RPM 패키지 전체 목록 확인 |
| `cat /proc/cpuinfo` | CPU 상세 정보 확인 |
