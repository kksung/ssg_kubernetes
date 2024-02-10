# (firststep - 한걸음) 3-tier 웹사이트
> Step 1 - Dockerizing
> > Step 2 - Kubernetes 배포
<img src="https://github.com/kksung/ssg_kubernetes/assets/110016279/c0d1c189-4ddd-406c-89e2-b964d9cafb5a" width=750 height=300>

[개발 페이지 참고](https://github.com/kksung/temp-firststep)

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

```
docker image build -t kksung/sql-db .
docker container run -d -p 3306 --name sqlcontainer -v sql:/var/lib/mysql --net=mybridgenetwork kksung/sql-db
```

<br>

2. Flask

```

```

<br>

3. React

```

```


<br>

## Kubernetes 배포

