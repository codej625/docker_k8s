# 도커 기본 명령어 모음

<br />
<br />

* 도커(Docker)와 관련된 기본 명령어를 정리한 가이드이다.

---

<br />
<br />
<br />
<br />

1. 기본 명령어

<br />

| 명령어               | 설명                                                                                           | 주요 옵션 및 예시                                |
|----------------------|-----------------------------------------------------------------------------------------------|----------------------------------------------|
| `docker build`       | 도커파일을 사용하여 이미지를 생성합니다.                                                         | `-t`: 이미지 이름 및 태그 지정<br>`docker build -t my-image:1.0 .` |
| `docker pull`        | 도커 허브 또는 다른 레지스트리에서 이미지를 다운로드합니다.                                       | `docker pull ubuntu:20.04`                   |
| `docker run`         | 이미지를 사용하여 새로운 컨테이너를 생성하고 실행합니다.                                          | `-d`: 백그라운드 실행<br>`-p`: 포트 매핑<br>`--name`: 컨테이너 이름 지정<br>`docker run -d -p 8080:80 --name my-nginx nginx` |
| `docker ps`          | 실행 중인 컨테이너 목록을 표시합니다. `-a`로 모든 컨테이너 확인 가능.                             | `-a`: 모든 컨테이너 표시<br>`docker ps -a`   |
| `docker stop`        | 실행 중인 컨테이너를 중지합니다.                                                                | `docker stop my-container`                   |
| `docker start`       | 중지된 컨테이너를 기존 설정대로 다시 시작합니다.                                                | `docker start my-container`                  |
| `docker rm`          | 중지된 컨테이너를 삭제합니다. `-f`로 실행 중인 컨테이너 강제 삭제 가능.                          | `-f`: 강제 삭제<br>`docker rm -f my-container` |
| `docker rmi`         | 로컬에 저장된 이미지를 삭제합니다. `-f`로 사용 중인 이미지 강제 삭제 가능.                       | `-f`: 강제 삭제<br>`docker rmi -f ubuntu:20.04` |
| `docker exec`        | 실행 중인 컨테이너 내에서 명령을 실행합니다.                                                   | `-it`: 인터랙티브 셸 실행<br>`docker exec -it my-container bash` |
| `docker logs`        | 컨테이너의 로그를 확인합니다. `-f`로 실시간 로그 확인 가능.                                       | `-f`: 실시간 로그<br>`docker logs -f my-container` |
| `docker images`      | 로컬에 저장된 이미지 목록을 표시합니다.                                                        | `docker images`                              |
| `docker inspect`     | 컨테이너 또는 이미지의 상세 정보를 확인합니다.                                                  | `docker inspect my-container`                |
| `docker network ls`  | 도커 네트워크 목록을 표시합니다.                                                               | `docker network ls`                          |
| `docker volume ls`   | 도커 볼륨 목록을 표시합니다.                                                                   | `docker volume ls`                           |

<br />

`추가 참고`

- 명령어 실행 환경: 일부 명령어는 컨테이너 ID 또는 이름이 필요하다. 컨테이너 ID는 `docker ps`로 확인 가능하다.
- 도커 허브: 공식 이미지는 `docker pull`로 다운로드하거나 `docker run` 시 자동으로 가져온다.
- 도커 컴포즈 설치: `docker-compose`는 별도 설치가 필요할 수 있으며, 최신 도커 버전에서는 `docker compose`(하이픈 없이)로 통합되었다.
