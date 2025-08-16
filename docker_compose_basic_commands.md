# 도커 컴포즈 (Docker Compose)

<br />
<br />

* docker-compose는 여러 컨테이너를 정의하고 관리하기 위한 도구이다.

---

```  
`docker-compose.yml` 파일을 사용해,
애플리케이션의 서비스, 네트워크, 볼륨 등을 정의한다.
```

<br />
<br />
<br />
<br />

1. 주요 명령어

<br />

| 명령어                    | 설명                                                                                           | 주요 옵션 및 예시                                |
|---------------------------|-----------------------------------------------------------------------------------------------|----------------------------------------------|
| `docker-compose up`       | `docker-compose.yml`에 정의된 서비스를 생성하고 시작합니다.                                      | `-d`: 백그라운드 실행<br>`docker-compose up -d` |
| `docker-compose down`     | 실행 중인 서비스를 중지하고 관련 리소스(컨테이너, 네트워크 등)를 삭제합니다.                      | `docker-compose down`                        |
| `docker-compose ps`       | 현재 프로젝트의 컨테이너 목록을 표시합니다.                                                     | `docker-compose ps`                          |
| `docker-compose logs`     | 현재 프로젝트의 컨테이너 로그를 확인합니다. `-f`로 실시간 로그 확인 가능.                         | `-f`: 실시간 로그<br>`docker-compose logs -f` |
| `docker-compose build`    | `docker-compose.yml`에 정의된 서비스의 이미지를 빌드합니다.                                      | `docker-compose build`                       |
| `docker-compose stop`     | 현재 프로젝트의 컨테이너를 중지합니다.                                                         | `docker-compose stop`                        |
| `docker-compose start`    | 중지된 컨테이너를 다시 시작합니다.                                                             | `docker-compose start`                       |
| `docker-compose rm`       | 중지된 컨테이너를 삭제합니다.                                                                 | `-f`: 강제 삭제<br>`docker-compose rm -f`    |

<br />

`참고`

* `docker-compose.yml` 파일은 서비스, 네트워크, 볼륨 등을 YAML 형식으로 정의한다.
* 예시: 웹 서버와 데이터베이스를 함께 실행하려면 `docker-compose.yml`에 두 서비스를 정의하고 `docker-compose up`으로 실행한다.
* 주요 옵션: `--build`(빌드 후 실행), `--scale`(컨테이너 수 조정).
