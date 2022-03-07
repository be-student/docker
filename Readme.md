# 기초 커맨드

docker run -p localport:containerport name wanttostart
docker run -it name 명령어 //-it는 여러 명령어 합치기 가능
//제일 뒤가 sh면 실행 이후에 계속 콘솔 치는 것 가능
못 했을 경우 exec으로 중간에 넣어줄 수도 있음

docker build -t name ./ 이름 규칙은 저장소명/이름:버전

docker ps -a
docker stop id//gracefully kill
docker kill id//kill
docker run -p 포트:포트 -v /usr/src/app/node_modules -v %cd%:/usr/src/app 아이디

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

# 현재까지 진행 방법

파일 작성 => 파일로 이미지 생성 =>이미지로 컨테이너 생성 후 앱 실행
핫 리로딩이 문제가 됨.
-d 가 콘솔창을 이후에 계속 쓸 수 있게 함.

FROM node:10
WORKDIR /usr/src/app
COPY package.json ./
RUN npm install
COPY ./ ./
CMD ["node","server.js"]
로 변경하는 것이 좋은 이유는 레이어 변경이 가장 끝에서만 되는 것이 좋기에 그럼

# Docker volume

volume으로 용량을 처리할 수가 있음
docker mapping
docker run -p 포트:포트 -v /usr/src/app/node_modules -v %cd%:/usr/src/app 아이디
%cd% 부분은 mac은 $(pwd) 이고, :를 기준으로 왼쪽을 참조한다는 느낌으로 보면 됨
host directory에 node_module가 없기에 컨테이너에 맵핑을 하지 않도록 함. 앞 부분의 역할.
뒷 부분은 pwd경로에 있는 디렉토리 혹은 파일을 /usr/src/app 경로에서 참조한다.

이미지 빌드 과정이 없어지는 장점이 있긴 하지만, 껐다 켜야 하는 것은 변하지 않는다는 점은 기억
