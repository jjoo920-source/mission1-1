# mission1-1
### attach vs exec 차이

| 구분 | docker attach | docker exec |
|------|--------------|-------------|
| 연결 대상 | 메인 프로세스(PID 1) | 새 프로세스 생성 |
| exit 후 컨테이너 | 종료(Exited) | 계속 실행(Running) |
| 실무 활용 | 드물게 사용 | 디버깅/점검 시 주로 사용 |

# 실습 명령
docker start 243f63f2727a
docker attach 243f63f2727a  # exit 후 컨테이너 종료 확인
docker start 243f63f2727a
docker exec -it 243f63f2727a bash  # exit 후 컨테이너 유지 확인