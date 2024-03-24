# 3-tier 웹 Dockerizing & Kubernetes 배포 개인프로젝트
> 개발 웹 = first-step, 한걸음
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/c0d1c189-4ddd-406c-89e2-b964d9cafb5a" width=750 height=300>

[개발 페이지 참고](https://github.com/kksung/ssg_webapi) -> HTML을 통한 프론트엔드 화면에서 React로 변경하여 프로젝트 수행

<br>

## 웹페이지 다이어그램 구성도
- 3-tier

<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/a335589e-089e-4eb3-8ed8-c2d0af3430eb" width=300 height=400>

## Dockerizing 개요
> 도커 이미지 빌드 -> 도커 컨테이너 실행

* 컨테이너가 시작할 때 사용할 네트워크 생성
  - MySQL, React, Flask 도커 컨테이너 각각 모두 같은 네트워크 상에 배치

<br>

- 도커 네트워크 생성 -> 'mybridgenetwork' 
```
docker network create --driver=bridge mybridgenetwork
docker network ls
```

<br>

## Dockerizing 순서
> 1 - MySQL (DB) 2 - Flask (Server) 3 - React

<br>

## Dockerizing 다이어그램 구성도
- 도커 컨테이너
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/064b3cc1-d98b-405d-a8c1-a8d6f884be39" width=400 height=450>


### MySQL DB Dockerizing
- DB : firststep
  - Table : user, board
- DB firststep data export -> ﻿c:\docker\init_db

<br>

- 파일 구조도
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/dc3fdce0-b907-4660-93f9-ea618d2c83cc" width=225 height=165>


- DB 도커 이미지 빌드 & 도커 컨테이너 실행 명령어
```
docker image build -t kksung/sql-db .
docker container run -d -p 3306 --name sqlcontainer -v sql:/var/lib/mysql --net=mybridgenetwork kksung/sql-db
```

<br>

### Flask Server Dockerizing
- 파일 구조도
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/3fa6d585-e948-477a-ae85-f2a7faf24da5" width=270 height=220>


- 빌드 전 포인트 -> app.py mysql 연결코드 'pymysql.connect' 수정


```
# host = mysql 도커 컨테이너 이름으로 변경
def getCon():
  return pymysql.connect(host="sqlcontainer",
                     user="root", password="passwd", 
                     db="firststep",
                     charset="utf8",
                     cursorclass=pymysql.cursors.DictCursor)
```

<br>

- Flask Server 도커 이미지 빌드 & 도커 컨테이너 실행 명령어
```
docker image build -t kksung/flask-server .
docker container run -d -p 5000 --name flaskserver --net=mybridgenetwork kksung/flask-server
```

<br>

### React Dockerizing
- 빌드 포인트 -> axios 부분 url 변경

<br>

- React 프론트엔드 도커 이미지 빌드 & 도커 컨테이너 실행 명령
```
docker image build -t kksung/react-client .
docker container run -d -p 80 --name reactweb --net=mybridgenetwork kksung/react-client .
```

<br>

## Kubernetes 배포
> 이미지 빌드 -> 도커 허브 -> YAML에 기입 후 배포

<br>

## Kubernetes 배포 다이어그램 구성도
- SVC, Deployment
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/07a93b93-cf98-48b5-9bea-7c52dd5ffcd4" width=600 height=430>


### Kubernetes YAML & 배포 포인트
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/b8dbbda0-5df3-4913-8bf5-3574bc67cd30" width=330 height=250>

1. ﻿DB는 로컬환경에서 사용한 이미지 그대로 deployment yaml에 넣어서 배포 

2. ﻿Flask 서버는 app.py의 pymysql.connect 'host IP' 값을 mysql service의 'ClusterIP'값으로 변경 -> 이미지 빌드 -> 도커 허브 -> 배포

3. r﻿eact는 Component의 js파일에서 'axios부분 path 값'을 'Flask 로드밸런서 서비스 External-IP 주소:5000'으로 작성 -> 이미지 빌드 -> 도커 허브 -> 배포

<br>

### NFS PV? 
- MySQL Deployment Volume으로 NFS PV를 사용, PVC를 통한 바인딩
- PV, PVC -> 파드가 볼륨의 세부적인 사항을 몰라도 볼륨을 사용할 수 있도록 추상화해 주는 역할
- NFS PV 사용 -> 클러스터 밖 외부 볼륨개념으로, 데이터 저장에 용이하다고 생각하여 사용
  - hostPath -> 노드에 저장

<br>

## 프로젝트 유의사항 & 개선사항
- Dockerizing - 컨테이너 모두 동일한 도커 네트워크를 사용해야함 -> 생성한 'mybridgenetwork' 사용 
- Dockerizing - 컨테이너 하나씩 순차적으로 실행해야하는 번거로움 -> 추후 docker-compose 파일 활용
- 코드의 연결 포인트 값을 배포할 때마다 수정해야하는 번거로움 -> 하드 코딩 대신 환경 변수 활용 
