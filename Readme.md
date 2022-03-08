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

# Docker Compose

다중 컨테이너 도커 애플리케이션을 실행하기 위한 도구, 쿠버네이틱스의 진화 전단계

# Redis

REmote Dictionary Server 메모리 기반 키, 값 구조 데이터 관리 시스템. 모든 데이터를 메모리에 저장하고, 빠르게 조회할 수 있는 nosql이다.
메모리에 저장하기에 mysql보다 빠르고, 메모리에 저장하더라도 영속적 보관이 가능. 서버 재부팅시 데이터 유지
일반 경우
const client=redis.createClient({
host:"https://redis-server.com",
port:6379
})
docker 환경의 경우 docker-compose.yml파일에 명시한 컨테이너 이름을 호스트에 넣어야 됨.
redisname",
port:6379

# 5장 docker compose

nodejs app + redis client / redis server
멀티 컨테이너 상황에서 네트워크 연결을 위한 것임

yaml 파일. yml 파일. YAML ain't markup language. xml이나 json포맷보다 조금 가독성이 높다는 특징
yml파일 구성 요소

```
version: "3" //버전
services: //컨테이너 정리
  redis-server: //컨테이너 이름
    image: "redis" //컨테이너 이미지
  node-app: //컨테이너 이름
    build: . //현 디렉토리에 있는 docker file 사용
    ports: //포트 매핑
      - "5000:8080"
```

docker-compose up vs docker-compose up --build
이미지가 없을 때 빌드하고 시작, 이미지가 있든 없든 빌드하고 시작 -d detach

# 6강 react 를 연동

개발 =>github, branch, =>travis CI =>test =>호스팅
docker file 작성시 dev인 경우 dockerfile.dev, prod인 경우dockerfile 을
이미지 빌드시 docker build -f Dockerfile.dev . 라는 거를 줘야 함. 그래야
-f 부분은 저 파일을 찾아라 라고 하는 것

docker run -p 3000:3000 -v /usr/src/app/node_modules -v %cd%:usr/src/app
node module은 없으니 맵핑하지 말고, 나머지는 현재 디렉토리에서 참고해라

docker run -it -p 3000:3000 -v /usr/src/app/node_modules -v %cd%:/usr/src/app presentsong/docker-react-app 이게 정말 너무 치기 귀찮음
docker.yml로 처리해버림

```
version: "3"
services:
  react://컨테이너 이름
    build: //현 디렉토리에 있는 도커파일 사용
      context: .  //도커이미지를 구성하기 위한 파일과 폴더들이 있는 위치
      dockerfile: Dockerfile.dev //도커 파일 이름 명시
    ports:
      - "3000:3000"
    volumes:
      - /usr/src/app/nodemodules //nodemodule는 참조하지 않겠다.
      - /./:/usr/src/app //나머지는 참조하겠다
    stdin_open: true
    environment: //핫 리로딩
      - CHOKIDAR_USEPOLLING=true
```

docker run -it 이름 npm run test
로 처리할 수 있음

운영 단계 : builder stage(빌드 파일 생성까지) run stage : 실제로 실행하는 과정

```
FROM node:alpine as builder //빌드 단계
WORKDIR '/usr/src/app' // /usr/src/app/build로 들어감. 생성된 파일들은 다
COPY package.json .
RUN npm install
COPY ./ ./
RUN npm run build

FROM nginx //run stage
EXPOSE 80
COPY --from=builder /usr/src/app/build /usr/share/nginx/html //builder에서 오는 파일을
//빌드 된 파일을 가져와서 어느 곳에다가 복사를 할지를 말함. 저 경로는 nginx가 제공할 수 있는 경로임.
//저렇게 안 한다면 nginx로 설정을 따로 해줘야 됨
```

# git에 push, travis ci 사용

travis ci 는 github repository 에서 자동 test, build, publish를 해줌.
local git=>github =>travis ci=>aws
순으로 이동이 됨.
