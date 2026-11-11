# Mission 1-1: 개발 워크스테이션 구축

## 1. 프로젝트 개요
터미널, Docker, Git을 직접 세팅하며 재현 가능한 개발 환경을 구축한다.
이미지/컨테이너 분리, 포트 매핑, 바인드 마운트, 볼륨 영속성을 직접 검증한다.
같은 서비스를 여러 번 실행해도 재현되는 환경을 만드는 사고방식을 경험하는 것이 목표이다.

---

## 2. 실행 환경

| 항목 | 버전/정보 |
|------|----------|
| OS | Windows 11 |
| Shell | Windows PowerShell |
| Docker | v29.9.3 |
| Git | v2.56.0.windows.4 |
| WSL2 | v3.0.5.0 |

---

## 3. 수행 항목 체크리스트

- [O] 터미널 기본 조작 (pwd, ls, mkdir, cd, touch, cat, cp, mv, rm)
- [O] 파일/디렉토리 권한 확인 및 변경 (chmod)
- [O] Docker 설치 및 기본 점검
- [O] Docker 기본 운영 명령 (images, ps, logs, stats)
- [O] hello-world / ubuntu 컨테이너 실행
- [O] Dockerfile 기반 커스텀 이미지 제작 (nginx)
- [O] 포트 매핑 및 브라우저 접속 확인
- [O] 바인드 마운트 변경 반영 확인
- [O] Docker 볼륨 영속성 검증
- [O] Git 설정 및 GitHub 연동

---

## 4. 터미널 조작 로그

```bash
# 현재 위치 확인
pwd
# C:\Users\010648577797779\Desktop\mission1-1

# 목록 확인 (숨김 파일 포함)
ls -al

# 디렉토리 생성 및 이동
mkdir test-dir
cd test-dir

# 파일 생성 및 내용 확인
touch test.txt
echo "hello" > test.txt
cat test.txt
# hello

# 내용 수정 확인
echo "gello" > test.txt
cat test.txt
# gello

# 복사, 이름변경, 삭제
cp test.txt test-copy.txt
mv test-copy.txt renamed.txt
rm renamed.txt
```

---

## 5. 권한 실습

```bash
# 파일 권한 확인 (초기 상태)
ls -l test.txt
# -rw-r--r-- (644)

# 777로 변경
chmod 777 test.txt
ls -l test.txt
# -rwxrwxrwx (777)

# 755로 변경
chmod 755 test.txt
ls -l test.txt
# -rwxr-xr-x (755)

# 디렉토리 권한 확인
ls -ld test-dir
# drwxr-xr-x (755)
```

### 권한 표기 해석
| 표기 | 의미 |
|------|------|
| 755 | 소유자: rwx / 그룹: r-x / 기타: r-x |
| 644 | 소유자: rw- / 그룹: r-- / 기타: r-- |
| 777 | 소유자: rwx / 그룹: rwx / 기타: rwx |

---

## 6. Docker 설치 및 기본 점검

```bash
# 버전 확인
docker --version
# Docker version 29.9.3

# 데몬 동작 확인
docker info
# Server Version: 29.9.3
# Storage Driver: overlay2
# ...

# 이미지 목록
docker images
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# nginx        latest    ...

# 전체 컨테이너 목록
docker ps -a

# 로그 확인
docker logs my-web

# 리소스 확인
docker stats --no-stream
```

---

## 7. 컨테이너 실행 실습

### hello-world
```bash
docker run hello-world
# Hello from Docker!
# This message shows that your installation appears to be working correctly.
```

### ubuntu 내부 진입
```bash
docker run -it ubuntu bash

# 컨테이너 내부 명령 실행
ls
# bin  boot  dev  etc  home  lib ...

pwd
# /

lsb_release -a
# Ubuntu 22.04 LTS

exit
```

### attach vs exec 차이
| 구분 | attach | exec |
|------|--------|------|
| 연결 대상 | 메인 프로세스(PID 1) | 새 프로세스 |
| exit 시 | 컨테이너 종료됨 | 컨테이너 계속 실행 |
| 용도 | 메인 로그 확인 | 추가 명령 실행/디버깅 |

---

## 8. Dockerfile 기반 커스텀 이미지

### 베이스 이미지: nginx:latest
### 커스텀 포인트: 정적 HTML 파일 교체

**app/index.html**
```html
<h1>Hello, Docker!</h1>
```

**Dockerfile**
```dockerfile
FROM nginx:latest
COPY app/ /usr/share/nginx/html
EXPOSE 80
```

```bash
# 이미지 빌드
docker build -t my-nginx .
# Successfully built xxxxxxxx
# Successfully tagged my-nginx:latest

# 컨테이너 실행 (포트 매핑)
docker run -d -p 8080:80 --name my-web my-nginx

# 접속 확인
curl http://localhost:8080
# <h1>Hello, Docker!</h1>
```

### 포트 매핑이 필요한 이유
> 컨테이너는 격리된 네트워크 환경에서 실행된다.
> `-p 8080:80`은 호스트의 8080 포트를 컨테이너의 80 포트에 연결하여
> 외부에서 컨테이너 내부 서비스에 접근할 수 있게 한다.

💻 브라우저 접속 화면: `http://localhost:8080` → "Hello, Docker!" 출력 확인

📸 브라우저 접속 화면:

![포트매핑 접속 확인](images/browser-8080.png)

---

## 9. 바인드 마운트

```bash
# 바인드 마운트로 컨테이너 실행
docker run -d -p 8081:80 --name my-bind \
  -v ${PWD}/app:/usr/share/nginx/html \
  my-nginx

# 호스트 파일 수정
echo "<h1>바인드 마운트 변경 확인!</h1>" > app/index.html

# 브라우저 새로고침 → 변경 즉시 반영 확인
curl http://localhost:8081
# <h1>바인드 마운트 변경 확인!</h1>
```

### 바인드 마운트 동작 원리
> 호스트의 디렉토리를 컨테이너 내부 경로에 직접 연결한다.
> 호스트 파일을 수정하면 컨테이너 재시작 없이 즉시 반영된다.

---

## 10. Docker 볼륨 영속성 검증

```bash
# 볼륨 생성
docker volume create my-vol
# my-vol

# 볼륨 목록 확인
docker volume ls
# DRIVER    VOLUME NAME
# local     my-vol

# 컨테이너 실행 + 데이터 쓰기
docker run -d --name vol-test \
  -v my-vol:/data ubuntu \
  sh -c "echo '볼륨 영속성 테스트' > /data/test.txt && sleep 3600"
# 8af1b42db9af268c955da7b26bb594ea2d9ac72fdad0267997d0bcb6a68e6aab

# 데이터 확인
docker exec vol-test cat /data/test.txt
# 볼륨 영속성 테스트 ✅

# 컨테이너 삭제
docker rm -f vol-test
# vol-test

# 새 컨테이너로 데이터 재확인
docker run --rm -v my-vol:/data ubuntu cat /data/test.txt
# 볼륨 영속성 테스트 ✅ (컨테이너 삭제 후에도 데이터 유지!)
```

### 볼륨 vs 바인드 마운트 비교
| 구분 | 볼륨 | 바인드 마운트 |
|------|------|--------------|
| 저장 위치 | Docker 관리 영역 | 호스트 지정 경로 |
| 주 용도 | 데이터 영속성 보장 | 개발 중 실시간 반영 |
| 컨테이너 삭제 후 | 데이터 유지 | 데이터 유지 |

---

## 11. Git 설정 및 GitHub 연동

```bash
# Git 사용자 정보 확인
git config --list
# user.name=jjoo920-source
# user.email=...
# core.defaultbranch=main

# 저장소 초기화 및 원격 연결
git init
git remote add origin https://github.com/jjoo920-source/mission1-1.git

# 커밋 및 Push
git add .
git commit -m "Mission 1-1: Docker 볼륨 영속성 검증 완료"
git push origin main
# b90de74..78a8d20  main -> main ✅
```

---

## 12. 트러블슈팅

### 🔧 Case 1: Merge Conflict 발생
| 항목 | 내용 |
|------|------|
| 문제 | `git push` 시 원격과 로컬 브랜치 충돌 발생 |
| 원인 가설 | GitHub에서 직접 README를 수정한 뒤 로컬에서도 수정하여 히스토리 불일치 |
| 확인 | `git status`로 충돌 파일 확인, 파일 내 `<<<<<<` 마커 발견 |
| 해결 | 충돌 파일 직접 편집 후 `git add .` → `git commit` → `git push` 재시도 성공 |

### 🔧 Case 2: docker exec 명령 오류
| 항목 | 내용 |
|------|------|
| 문제 | `docker exec vol-test cat /data/test.txt` 실행 시 컨테이너를 찾을 수 없다는 오류 |
| 원인 가설 | `sleep 3600` 없이 컨테이너를 실행하여 명령 완료 즉시 컨테이너가 종료됨 |
| 확인 | `docker ps -a`로 컨테이너 상태 확인 → Exited 상태 |
| 해결 | `sh -c "echo '...' > /data/test.txt && sleep 3600"` 으로 컨테이너 유지 후 재실행 성공 |

---

## 13. 핵심 개념 정리

### 절대 경로 vs 상대 경로
| 구분 | 예시 | 설명 |
|------|------|------|
| 절대 경로 | `/usr/share/nginx/html` | 루트(/)부터 시작하는 전체 경로 |
| 상대 경로 | `./app` | 현재 위치 기준 경로 |

### Git vs GitHub
| 구분 | 역할 |
|------|------|
| Git | 로컬 버전 관리 도구 |
| GitHub | 원격 저장소 + 협업 플랫폼 |