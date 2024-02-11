# (firststep - 한걸음) 3-tier 웹사이트
> Step 1 - Dockerizing
> > Step 2 - Kubernetes 배포
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/c0d1c189-4ddd-406c-89e2-b964d9cafb5a" width=750 height=300>

[개발 페이지 참고](https://github.com/kksung/webapi) -> HTML을 통한 프론트엔드 화면에서 React로 변경하여 프로젝트 수행 

<br>

## Dockerizing

* 컨테이너가 시작할 때 사용할 네트워크 생성
  - MySQL, React, Flask 모두 같은 network로 컨테이너 실행! 
```
docker network create --driver=bridge mybridgenetwork # mybridgenetwork 이름의 network 생성
docker network ls
```

1. MySQL DB - firststep / Table - User, Board
   - export ﻿-> DB firststep -> ﻿c:\docker\init_db

- 파일 구조도

```
docker image build -t kksung/sql-db .
docker container run -d -p 3306 --name sqlcontainer -v sql:/var/lib/mysql --net=mybridgenetwork kksung/sql-db
```

<br>

2. Flask
- 파일 구조도
- 빌드 포인트
  - pymysql 연결 
```
docker image build -t kksung/flask-server .
docker container run -d -p 5000 --name flaskserver --net=mybridgenetwork kksung/flask-server
```

<br>

3. React
- 빌드 포인트
  - axios 부분 url 변경
```
docker image build -t kksung/react-client .
docker container run -d -p 80 --name reactweb --net=mybridgenetwork kksung/react-client .
```

<br>

## Kubernetes 배포
> 이미지 빌드 -> 도커 허브 -> YAML 이미지에 기입 후 배포

1. ﻿DB는 로컬환경에서 사용한 이미지 deployment에 넣어서 바로 배포

2. ﻿Flask 서버는 host IP 값을 mysql service의 ‘ClusterIP’로 변경

3. r﻿eact는 Component의 js파일에서 axios부분 Flask 로드밸런서 External-IP 주소:5000으로 작성
