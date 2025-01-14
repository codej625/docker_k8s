# Docker Compose

<br />
<br />

* Docker compose
---

```
Compose는 여러 개의 Docker 컨테이너를 정의하고, 설정하여 함께 실행할 수 있게 도와주는 도구이다.

docker-compose.yml 파일을 사용하면 여러 컨테이너를 한 번에 설정하고 실행할 수 있기 때문에, 
프로젝트에서 여러 서비스 를 하나의 YAML 파일로 정의할 수 있다.
(데이터베이스, 애플리케이션 서버 등)
```

<br />
<br />
<br />
<br />

1. 단일 컨테이너 실행과 차이점

| **항목**                    | **단일 Docker 컨테이너 실행 (`docker run`)**         | **Docker Compose (`docker-compose.yml`)**                |
|-----------------------------|---------------------------------------------------|-------------------------------------------------------|
| **목표**                     | 단일 컨테이너 실행                                  | 여러 컨테이너(서비스) 정의 및 동시에 실행                |
| **사용 명령어**              | `docker run`                                      | `docker-compose up`                                    |
| **설정 파일**                | 설정 파일 없음 (명령어에 모든 설정 포함)            | `docker-compose.yml` 파일에 모든 설정 포함               |
| **컨테이너 수**               | 하나의 컨테이너만 실행                              | 여러 개의 컨테이너(서비스)를 동시에 실행                |
| **서비스 간 연결**            | 수동으로 `docker network`로 연결                    | `docker-compose.yml`에서 서비스 간 네트워크 연결 자동 관리 |
| **환경 설정 관리**            | 환경 변수, 포트, 볼륨 설정은 `docker run` 명령어에 포함 | 환경 변수, 포트, 볼륨 등을 `docker-compose.yml`에 정의 |
| **네트워크 설정**             | 기본 네트워크 사용 (명시적으로 설정하지 않으면 자동) | 네트워크를 명시적으로 설정하여 서비스 간 연결 관리 가능  |
| **서비스 종속성 관리**         | 수동으로 서비스 간 종속성 관리 (예: 컨테이너 시작 순서) | `depends_on`을 사용하여 서비스 시작 순서 및 의존성 관리 |
| **컨테이너 실행 상태 관리**    | 개별 컨테이너 상태 관리 필요 (컨테이너 하나하나 직접 관리) | `docker-compose`로 여러 컨테이너를 한 번에 관리         |
| **재시작 옵션**               | `--restart` 옵션을 사용하여 개별 컨테이너의 재시작 관리 | `restart` 옵션을 `docker-compose.yml`에서 설정          |
| **확장성 및 복잡성**          | 여러 컨테이너를 실행하려면 각각 `docker run` 명령어 실행 | 여러 컨테이너가 필요한 복잡한 애플리케이션을 쉽게 설정 |
| **실행 예시**                 | `docker run -d -p 3000:3000 --name app node:16`       | `docker-compose up` (정의된 모든 서비스 실행)          |

<br />
<br />

2. Command

```
1) docker-compose up
docker-compose.yml 파일에 정의된 서비스를 실행한다.

옵션 -d: docker-compose.yml에 정의된 서비스를 백그라운드에서 실행
    --build: 이미 빌드된 이미지가 없거나, 이미지가 변경된 경우 새로 빌드하고 실행한다.
```

```
2) docker-compose down
실행 중인 컨테이너들을 종료하고 서비스에 의해 생성된 네트워크를 삭제한다.

옵션 -v: 실행 중인 서비스의 컨테이너뿐만 아니라 연결된 볼륨까지 모두 삭제하여 환경을 완전히 초기화
```

```
3) docker-compose ps
실행 중인 컨테이너의 상태를 확인한다.
```

```
4) docker-compose logs
실행 중인 서비스의 로그를 확인한다.

옵션 docker-compose logs db: 특정 서비스의 로그만 보고 싶으면 서비스 이름을 추가할 수 있다.
```

```
5) docker-compose restart
실행 중인 서비스를 재시작한다.

옵션 docker-compose restart db: 특정 서비스만 재시작하려면 서비스 이름을 추가한다.
```

<br />
<br />

3. docker-compose 작성 예시

```yaml
# docker-compose.yml

version: '3.1'

services:
  db:
    image: postgres:15.10 # 이미지가 없으면 Docker Hub애서 자동으로 다운 받는다.
    restart: no # 여러 가지 옵션이 있다. no는 기본값으로 컨테이너가 종료되면 재 시작하지 않는다.
    env_file:
      - ./.env # 환경 변수가 있는 파일을 읽어온다.
    ports:
      - 5432:5432 # 왼쪽은 실제 매핑되는 포트, 오른쪽은 현재의 포트
    volumes: # 안 적으면 기본 Docker의 저장 위치에 저장 된다.
      - ./db:/var/lib/postgresql/data # Volume의 저장 위치
    environment: # 안 적으면 기본 사용자는 posrgres
      POSTGRES_USER: {user}
      POSTGRES_PASSWORD: {password}
      POSTGRES_DB: {db_name}
```

```
// .env

POSTGRES_DB={data_base_name}
POSTGRES_USER={user}
POSTGRES_PASSWORD={password}

PostgreSQL 컨테이너가 시작되면,
{data_base_name}라는 이름의 데이터베이스가 생성된다.

즉, PostgreSQL의 초기 사용자 계정과 데이터베이스를 자동으로 생성하여,
컨테이너가 처음 시작될 때 바로 사용할 수 있게 해준다.
```

<br />
<br />

4. 정리

```
단일 Docker 컨테이너 실행은 한 번에 하나의 서비스(컨테이너)를 실행하는 방식이다.

Docker Compose는 여러 개의 서비스를 동시에 정의하고 실행하는 데 유용한 도구로,
여러 컨테이너 간의 종속성도 관리할 수 있다.
```
