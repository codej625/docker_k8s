# k3s

<br />
<br />

* k8s? k3s?

---

```
k3s란 쿠버네티스(k8s)의 경량 버전이다.

설치가 간단하며, 컴퓨팅 리소스를 훨씬 적게 먹는다.

그리고 실제 프로덕션 환경에서 종종 사용하기도 한다.
```

<br />
<br />
<br />
<br />

1. 추천 사양

```
k3s의 권장 설치 사양이 CPU 2 core, RAM 1 GB이다.

백엔드 서버도 같이 실행시킬 것을 고려해 AWS EC2 인스턴스는 t4g.small로 생성하면 좋다.
```

<br />
<br />
<br />

2. Docker 설치

```
$ sudo apt-get update && \
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo apt-key fingerprint 0EBFCD88 && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update && \
sudo apt-get install -y docker-ce && \
sudo usermod -aG docker ubuntu && \
newgrp docker && \
sudo curl -L "https://github.com/docker/compose/releases/download/2.27.1/docker-compose-$(uname -s)-$(unam
sudo chmod +x /usr/local/bin/docker-compose && \
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

<br />
<br />
<br />

3. 설치 확인

```
$ docker -v # Docker 버전 확인
$ docker compose version # Docker Compose 버전 확인
```

<br />
<br />
<br />

4. k3s 설치하기

```
$ curl -sfL https://get.k3s.io | sh - # k3s 설치
$ sudo chmod 644 /etc/rancher/k3s/k3s.yaml # 권한 부여
$ sudo kubectl version # k3s 잘 설치됐는 지 확인
```
