docker run -p localport:containerport name wanttostart
docker run -it name 명령어 //-it는 여러 명령어 합치기 가능
//제일 뒤가 sh면 실행 이후에 계속 콘솔 치는 것 가능

docker build -t name ./ 이름 규칙은 저장소명/이름:버전

docker ps -a
docker stop id//gracefully kill
docker kill id//kill

# 기존 가상화

하드디스크=>window=>os=>프로그램
docker : 하드디스크 =>window=>linux=>program linux 하나 위에서 모든 것을 처리함

# basic things

docker harddisk,network,cpu,ram..=>window=>linux=>containuer
docker =>cgroup(프로그램의 리소스 활용 제한),namespace(격리 도와줌)
working directory

# image

dockerfile=>client=>server=>image
시작 명령어 =>커널 => 하드디스크에 스냅샷

# work directory 를 쓰는 이유

원래 파일 시스템에 있는 기본 파일과 덮어씌워지는 경우가 생길 수 있음
bin, lib같은 것들
정리가 매우 어려움
기본 파일은 계속 /에 있지만 work dir에 기본적인 것들을 다 복사하기에 조금 정리가 잘 됨
