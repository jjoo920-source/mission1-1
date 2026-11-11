# Mission 1-1: 개발 워크스테이션 구축

## 1. 프로젝트 개요
터미널, Docker, Git을 직접 세팅하며 재현 가능한 개발 환경을 구축한다.
이미지/컨테이너 분리, 포트 매핑, 바인드 마운트, 볼륨 영속성을 직접 검증한다.
같은 서비스를 여러 번 실행해도 재현되는 환경을 만드는 사고방식을 경험하는 것이 목표이다.

---

## 2. 프로젝트 디렉토리 구조

    mission1-1/
    ├── docker-web/          # Nginx 커스텀 이미지 빌드 실습 폴더
    │   ├── Dockerfile       # nginx 이미지 빌드 설정 파일
    │   └── app/             # nginx에 배포할 정적 HTML 파일
    │       └── index.html
    ├── images/              # 실습 증거 스크린샷 보관
    ├── pratice/             # 명령어 연습용 임시 파일 저장
    └── README.md            # 프로젝트 전체 문서

### 폴더 역할 설명
| 폴더 | 역할 |
|------|------|
| docker-web/ | Dockerfile 및 nginx 서비스 파일 관리 |
| docker-web/app/ | 컨테이너에 복사될 정적 웹 파일 보관 |
| images/ | 브라우저 접속·명령 실행 캡처 스크린샷 저장 |
| pratice/ | 터미널 명령어 연습용 임시 파일 저장 |

---

## 3. 실행 환경

| 항목 | 버전/정보 |
|------|----------|
| OS | Windows 11 + WSL2 |
| Shell | Bash (WSL2) |
| Docker | v29.9.3 |
| Git | v2.56.0.windows.4 |
| WSL2 | v3.0.5.0 |

---

## 4. 수행 항목 체크리스트

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

## 5. 터미널 조작 로그

    # 현재 위치 확인
    pwd
    # C:\Users\[USERNAME]\Desktop\mission1-1
    # ⚠️ 경로 중 [USERNAME] 부분은 본인의 Windows 사용자명으로 대체하세요.

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

---

## 6. 권한 실습

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

### 권한 표기 해석
| 표기 | 의미 |
|------|------|
| 755 | 소유자: rwx / 그룹: r-x / 기타: r-x |
| 644 | 소유자: rw- / 그룹: r-- / 기타: r-- |
| 777 | 소유자: rwx / 그룹: rwx / 기타: rwx |

### 숫자 권한 계산 방법
| 문자 | 의미 | 숫자 값 |
|------|------|---------|
| r | 읽기 (read) | 4 |
| w | 쓰기 (write) | 2 |
| x | 실행 (execute) | 1 |
| - | 권한 없음 | 0 |

> 예시
> rwx = 4 + 2 + 1 = 7
> r-x = 4 + 0 + 1 = 5
> rw- = 4 + 2 + 0 = 6
> 755 = 소유자(rwx=7) / 그룹(r-x=5) / 기타(r-x=5)

---

## 7. Docker 설치 및 기본 점검

    # 버전 확인
    docker --version
    # Docker version 29.9.3

    # 데몬 동작 확인
    docker info
    # Server Version: 29.9.3
    # Storage Driver: overlay2

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

---

## 8. 컨테이너 실행 실습

### hello-world

    docker run hello-world
    # Hello from Docker!
    # This message shows that your installation appears to be working correctly.

### ubuntu 내부 진입

    docker run -it ubuntu bash

    # 컨테이너 내부 명령 실행
    ls
    # bin  boot  dev  etc  home  lib ...

    pwd
    # /

    lsb_release -a
    # Ubuntu 22.04 LTS

    exit

### attach vs exec 차이
| 구분 | attach | exec |
|------|--------|------|
| 연결 대상 | 메인 프로세스(PID 1) | 새 프로세스 |
| exit 시 | 컨테이너 종료됨 | 컨테이너 계속 실행 |
| 용도 | 메인 로그 확인 | 추가 명령 실행/디버깅 |

---

## 9. Dockerfile 기반 커스텀 이미지

베이스 이미지: nginx:latest
커스텀 포인트: 정적 HTML 파일 교체

app/index.html

    <h1>Hello, Docker!</h1>

Dockerfile

    FROM nginx:latest
    COPY app/ /usr/share/nginx/html
    EXPOSE 80

    # 이미지 빌드
    docker build -t my-nginx .
    # Successfully built xxxxxxxx
    # Successfully tagged my-nginx:latest

    # 컨테이너 실행 (포트 매핑)
    docker run -d -p 8080:80 --name my-web my-nginx

    # 접속 확인
    curl http://localhost:8080
    # <h1>Hello, Docker!</h1>

### 포트 매핑이 필요한 이유
> 컨테이너는 격리된 네트워크 환경에서 실행된다.
> -p 8080:80 은 호스트의 8080 포트를 컨테이너의 80 포트에 연결하여
> 외부에서 컨테이너 내부 서비스에 접근할 수 있게 한다.

### Docker 브리지 네트워크 구조

    호스트 (Windows)
    │
    ├── 8080 포트 (외부 접속)
    │       │  -p 8080:80 매핑
    │       ▼
    │   [docker0 브리지 네트워크]
    │       │
    │       └── 컨테이너 내부 80 포트 (nginx)

> Docker는 기본적으로 bridge 네트워크를 사용한다.
> 컨테이너는 이 가상 네트워크 안에 격리되어 있으며
> -p 옵션으로 호스트 포트와 연결해야만 외부에서 접근 가능하다.

브라우저 접속 화면:
![포트매핑 접속 확인](images/browser-8080.png)

---

## 10. 바인드 마운트

    # 바인드 마운트로 컨테이너 실행
    docker run -d -p 8081:80 --name my-bind \
      -v ${PWD}/app:/usr/share/nginx/html \
      my-nginx

    # 호스트 파일 수정
    echo "<h1>바인드 마운트 변경 확인!</h1>" > app/index.html

    # 변경 반영 확인
    curl http://localhost:8081
    # <h1>바인드 마운트 변경 확인!</h1>

### 바인드 마운트 동작 원리
> 호스트의 디렉토리를 컨테이너 내부 경로에 직접 연결한다.
> 호스트 파일을 수정하면 컨테이너 재시작 없이 즉시 반영된다.

---

## 11. Docker 볼륨 영속성 검증

    # 볼륨 생성
    docker volume create my-vol
    # my-vol

    # 볼륨 목록 확인
    docker volume ls
    # DRIVER    VOLUME NAME
    # local     my-vol

    # 볼륨 상세 정보 확인
    docker volume inspect my-vol
    # [
    #   {
    #     "Name": "my-vol",
    #     "Driver": "local",
    #     "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
    #     "Scope": "local"
    #   }
    # ]

    # 컨테이너 실행 + 데이터 쓰기
    docker run -d --name vol-test \
      -v my-vol:/data ubuntu \
      sh -c "echo '볼륨 영속성 테스트' > /data/test.txt && sleep 3600"

    # 데이터 확인
    docker exec vol-test cat /data/test.txt
    # 볼륨 영속성 테스트 ✅

    # 컨테이너 삭제
    docker rm -f vol-test
    # vol-test

    # 새 컨테이너로 데이터 재확인
    docker run --rm -v my-vol:/data ubuntu cat /data/test.txt
    # 볼륨 영속성 테스트 ✅ (컨테이너 삭제 후에도 데이터 유지!)

### 볼륨 vs 바인드 마운트 비교
| 구분 | 볼륨 | 바인드 마운트 |
|------|------|--------------|
| 저장 위치 | Docker 관리 영역 | 호스트 지정 경로 |
| 주 용도 | 데이터 영속성 보장 | 개발 중 실시간 반영 |
| 컨테이너 삭제 후 | 데이터 유지 | 데이터 유지 |

## 11-1. 절대경로 vs 상대경로 선택 기준

### 개념 비교
| 구분 | 형식 예시 | 설명 |
|------|-----------|------|
| 절대경로 | `/home/user/app` | 루트(/)부터 시작하는 전체 경로 |
| 상대경로 | `${PWD}/app` | 현재 작업 디렉토리 기준 경로 |

### 장단점 비교
| 구분 | 절대경로 | 상대경로 |
|------|----------|----------|
| 장점 | 경로가 명확하여 오류 없음 | 환경이 달라도 유연하게 동작 |
| 단점 | 다른 환경에서 경로 수정 필요 | 실행 위치에 따라 경로 달라짐 |
| 사용처 | 컨테이너 내부 경로 지정 시 | 로컬 개발 환경 볼륨 마운트 시 |

### 선택 기준
| 상황 | 권장 방식 | 이유 |
|------|-----------|------|
| 컨테이너 내부 경로 | ✅ 절대경로 | 컨테이너 내부는 항상 동일한 경로 구조 |
| 로컬 볼륨 마운트 | ✅ 상대경로(`${PWD}`) | 팀원마다 로컬 경로가 다를 수 있음 |
| 스크립트 배포 자동화 | ✅ 절대경로 | 실행 위치와 무관하게 안정적 동작 |
| 개발 중 테스트 | ✅ 상대경로 | 빠르게 경로 변경 가능 |

### 본 프로젝트 적용 예시
# 상대경로 사용 (로컬 개발 환경)
docker run -d \
  -v ${PWD}/app:/usr/share/nginx/html \  ← 로컬 경로: 상대경로
  -p 8080:80 \
  --name my-web my-nginx

# 컨테이너 내부 경로는 절대경로 고정
/usr/share/nginx/html  ← 컨테이너 내부: 절대경로

---

## 12. Git 설정 및 GitHub 연동

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

---

## 13. 포트 충돌 진단 및 해결

### 포트 사용 프로세스 확인

    # Windows PowerShell
    netstat -ano | findstr :8080

    # 출력 예시
    # TCP    0.0.0.0:8080    0.0.0.0:0    LISTENING    1234
    #                                                   ↑ PID

### 포트 점유 프로세스 종료

    # 위에서 확인한 PID로 프로세스 강제 종료
    taskkill /PID 1234 /F

### Docker 컨테이너 포트 충돌 시

    # 실행 중인 컨테이너 확인
    docker ps

    # 충돌 컨테이너 중지 및 제거
    docker stop my-web
    docker rm my-web

    # 포트 해제 후 재실행
    docker run -d -p 8080:80 --name my-web my-nginx

---

## 14. 트러블슈팅

### Case 1: Merge Conflict 발생
| 항목 | 내용 |
|------|------|
| 문제 | git push 시 원격과 로컬 브랜치 충돌 발생 |
| 원인 가설 | GitHub에서 직접 README를 수정한 뒤 로컬에서도 같은 파일을 수정하여 충돌 발생 |
| 해결 | git pull --rebase origin main 으로 원격 변경사항을 먼저 적용 후 push |
| 결과 | 충돌 해소 및 push 성공 ✅ |

### Case 2: GitHub 대소문자 파일명 충돌
| 항목 | 내용 |
|------|------|
| 문제 | 로컬에서 README.md → readme.md 로 변경했으나 GitHub에 반영 안 됨 |
| 원인 | Windows 파일시스템은 대소문자 구분 없음 (case-insensitive) |
| 해결 | git rm --cached readme.md 로 Git 추적 해제 후 README.md 로 재등록 |
| 결과 | GitHub에 README.md 로 정상 반영 ✅ |

    # 실행 명령 순서
    git rm --cached readme.md
    git add README.md
    git commit -m "fix: README.md 대소문자 파일명 충돌 해결"
    git push origin main

### Case 3: 포트 충돌로 컨테이너 실행 실패
| 항목 | 내용 |
|------|------|
| 문제 | docker run -p 8080:80 실행 시 포트 이미 사용 중 오류 발생 |
| 원인 | 이전에 실행한 컨테이너가 8080 포트를 점유 중 |
| 해결 | docker ps 로 충돌 컨테이너 확인 후 stop/rm, 또는 다른 포트로 재실행 |
| 결과 | 컨테이너 정상 실행 ✅ |

    # 충돌 컨테이너 확인 및 제거
    docker ps
    docker stop my-web
    docker rm my-web

    # 포트 변경 후 재실행
    docker run -d -p 8082:80 --name my-web my-nginx

---

## 15. 참고 자료

- Docker 공식 문서: https://docs.docker.com
- Git 공식 문서: https://git-scm.com/doc
- nginx 공식 이미지: https://hub.docker.com/_/nginx
- Linux 권한 계산: https://chmod-calculator.com