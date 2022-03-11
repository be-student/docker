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
테스트 수행 준비(도커 환경 실행=>traivs ci도 도커로, dockerfile.dev로 이미지 생성)=> 수행(test 수행 방법 설정 .yml파일) =>aws로 배포(배포 방식도 설정)

sudo =>관리자 권한
language => 언어(플랫폼) 선택
services => 도커환경 구성
before install => 스크립트 실행할 수 있는 환경
script => 실행할 스크립트 설정
after_success => 테스트 성공 후 할 일

# ec2, elastic beanstalk

ec2는 컴퓨터 1대 임대와 유사함, elastic beanstalk는 ec2, db, security group, auto-scaling group, load balancer 가 함께 있는 복합체. apahe, nginx같은 서버부터 java, net, php, nodejs, python, go, docker같은 방식으로 개발된 웹 응용 프로그램을 배포할 때 확장하기 쉬운 서비스
로드 밸런서 : 알아서 요청 트래픽 서버별로 분배.
트래픽이 적을 때 : 브라우저 =>aws elastic beanstalk(로드밸런서 => ec2인스턴스(도커 컨테이너(어플리케이션)))
deploy 부분
provider : elasticbeanstalk, firebase, s3등 여러가지 곳에 다 제공이 가능함
region : aws 서비스가 위치한 물리적 장소
app : 이름
env : 이름을 써주면 됨
bucker_name : s3를 쓰는 경우 버킷 이름
bucker_name : 어플리케이션 이름과 동일
on
branch :어떤 branch에 push할 때 aws에 배포를 할 지
travis ci 에서 aws에서 실제로 소통할 수 있도록 인증 세팅.

secret key를 받는 방법
IAM(identity and access management)
IAM으로 인증, 로그인 권한 부여
처음 로그인 시 root사용자로 모든 리소스를 가능하지만, 루트 사용자에서 IAM에 권한을 부여하는 방법
iam 에서 사용자 추가
비밀 access key는 travis.yml에 있는 것에 바로 올리는 것이 아닌, 숨겨야 함. .env 사용 느낌
travis ci 사이트에서 dashboard, repository, more options, setting, environment variables.
access_key는 그냥 key, secret key는 secret key 에 있음.

# node 서버

nginx 프록시를 이용한 설계 vs nginx 정적 파일 제공 설계
![image](https://user-images.githubusercontent.com/80899085/157557417-65274b7e-1401-49c0-a437-fb4755e32963.png)
client => nginx(proxy) =>프론트(nginx 설정), 백엔드

default.conf
nginx 설정이 여기에 있음
/가 있으면 front로, api가 있으면 백으로 감.

# react 서버

```
server {
    listen 3000;

    location / {
      //경로에다가 build파일을 넣어둔다면
        root /usr/share/nginx/html; //html이 위치할 설정.

        index index.html index.htm; //index.html 을 index 페이지로 하겠다.

        try_files $uri  $uri/ /index.html; //라우팅할 때 필요함.
        //react는 spa임. 따라서 url/home에 접근한다고 했을 때, /home에 파일이 없다면 /home으로 index.html에서 라우팅 시켜준다.

    }
}
```

```
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf 이 부분은 우리가 작성한 default.conf를 nginx 설정으로 덮어 씌워주는 것
COPY --from=builder /app/build  /usr/share/nginx/html //빌드 파일을 제공할 수 있도록 옮겨 적어주는 것 builder stage에서 옴.
```

# db 구성

개발 : 도커 환경 이용
운영 환경 : AWS RDS 서비스 이용

개발
elastic beanstalk
client=>nginx=>프론트(nginx=>js,html,css, 3000번),백(server 5000번)=>mysql

운영
elastic beanstalk
client=>nginx=>프론트(nginx=>js,html,css, 3000번),백(server 5000번)
RDS로 mysql 부분 분리

db사용을 위해서는 db 설치파일을 이용해 설치, 노드 앱에 연결해야 하지만 그냥 노드 앱에 연결만 시켜줬습니다
이 부분을 도커 이미지를 이용해 설치를 함.

my.cnf 파일에 한글 utf8인코딩 설정을 해줘야 됨.

docker file에 add ./my.cnf 를 해야 함.
.dev는 굳이 달라질 이유가 없으면 그냥 똑같이 함.

nginx를 위한 도커 파일. proxy nginx를 위한 것임.
api 라면 nodejs로, 그냥은 정적 파일로.

front 안에 nginx가 있고, 그냥 전체 nginx가 있음.

```
upstream frontend {
    server frontend:3000;
}

upstream backend {
    server backend:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://frontend;
    }

    location /api {
        proxy_pass http://backend;
    }

    location /sockjs-node {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

}
```

프론트가 3000에 돌고 있다는 거를 그냥 적어둘 뿐임 그 이름은 dockere compose 파일에 있음.
server{
listen 80; 이 부분이 진짜 제일 바깥 서버 포트.
}
docker compose 파일에 적혀있는 것임. 전부 다 그 이름 그대로.
docker환경이 아니라면 전부 다 ip를 적어야 함. 하지만 도커기에 다 됨
밑부분은 개발 환경 react 에러임.

nginx를 위한 docker 파일 제작이 필요함.
nginx에 copy로 덮어 씌우는 것

docker 를 각각 만든다고 해도, docker-compose로 합쳐야 함.
그렇게 하지 않으면 어차피 통신이 안 됨.

docker-compose
volume 으로 node_modules 부분을 필요 없이 해버림, frontend 부분을 알아서 참조하도록 조금 세팅을 해줌. stdin_open:true react앱 종료시 생기는 버그를 잡아 줌.

nginx : restart부분 "no", always, on-failure unless-stopped : 가 있음.
unless-stopped : 개발자가 멈추고자 할 때 제외하면 계속 재시작

volume이 호스트 파일 시스템에 mysql db를 저장하고, 이를 참조하기에 이미지, 컨테이너 삭제시에도 데이터가 유지됨.
