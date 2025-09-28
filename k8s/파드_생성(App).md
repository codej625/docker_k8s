# 백엔드(Spring Boot) 서버를 파드(Pod)로 띄워보기

<br />
<br />

* App Server를 띄우기 위한 여러가지 준비

---

```
이번에는 스프링 서버를 파드로 띄어볼것이다.

다른 프레임워크와 방법이 크게 다르지 않으니
각자 사용하는 언어에 맞춰 응용 해보자.
```

<br />
<br />
<br />
<br />

1. Spring Boot 프로젝트 셋팅

```
https://start.spring.io 사이트에 접속하고,
스프링부트 프로젝트를 만들어준다.

Dependencies는 "Spring Web", "Spring Boot DevTools"를 추가한다.
```

<br />
<br />
<br />

2. 간단한 코드 작성

<br />

`AppController`

```java
@RestController
public class AppController {
  @GetMapping("/")
  public String home() {
    return "Hello, World!";
  }
}
```

<br />
<br />
<br />

3. 프로젝트 실행시켜보기 (+빌드)

```
./gradlew clean build 커맨드를 사용하면 jar 파일이 만들어진다.

java -jar [jar_name] 명령어로 작동을 확인해보자.

기본 포트는 8080이므로, 브라우저에 localhost:8080으로 접속하면 된다.
```

<br />
<br />
<br />

4. Dockerfile 작성하기

<br />

`Dockerfile`

```dockerfile
# 슬림 버전의 오픈JDK 17 이미지 사용
FROM openjdk:17-jdk-slim

# 필요한 도구들 설치 (curl, ps, netstat 등)
RUN apt-get update && apt-get install -y curl procps net-tools && rm -rf /var/lib/apt/lists/*

# 작업 디렉토리 설정
WORKDIR /app

# JAR 파일 복사 (파일을 복사 후 app.jar 이라는 이름으로 변경하고, 작업 디렉토리에 위치한다.)
COPY ./build/libs/spring-0.0.1-SNAPSHOT.jar app.jar

# 애플리케이션 실행 (java -jar app.jar 형태로 명령어가 작동한다.)
ENTRYPOINT ["java", "-jar", "app.jar"]
```

<br />
<br />
<br />

5. Dockerfile을 바탕으로 이미지 빌드하기

```
# 여기서 . 은 = /Users/codej625/Desktop/spring 이런 뜻이다.
$ docker build -t spring-server .
```

<br />
<br />
<br />

6. Docker Image 확인

```
docker image ls
혹은
docker images
```

<br />
<br />
<br />

7. 매니페스트 파일 작성하기

<br />

`spring-pod.yaml`

```yaml
apiVersion: v1
kind: Pod

metadata:
name: spring-pod

spec:
  containers:
    - name: spring-container
    image: spring-server
    ports:
      - containerPort: 8080
```

<br />
<br />
<br />

8. 매니페스트 파일을 기반으로 파드(Pod) 생성하기

```
# -f 는 file을 뜻한다.
$ kubectl apply -f spring-pod.yaml
```

<br />
<br />
<br />

9. 파드(Pod)가 잘 생성됐는 지 확인

```
# 모든 pod를 확인
kubectl get pods
```

```
STATUS를 보면,
ImagePullBackOff라고 떠 있는 것을 확인 할 수 있다.

이유는 이미지 풀 정책 때문이다.
```

<br />

`이미지 풀 정책 (Image Pull Policy)`

```
이미지 풀 정책(Image Pull Policy)이란 쿠버네티스가 yaml 파일을 읽어들여 파드를 생성할 때,
이미지를 어떻게 Pull을 받아올 건지에 대한 정책을 의미한다.
```

<br />

| 정책 | 설명 |
|-----|-----|
| Always | 로컬에서 이미지를 가져오지 않고, 무조건 레지스트리(= Dockerhub, ECR과 같은 원격 이미지 저장소)에서 가져온다. |
| IfNotPresent | 로컬에서 이미지를 먼저 가져온다. 만약 로컬에 이미지가 없는 경우에만 레지스트리에서 가져온다. |
| Never | 로컬에서만 이미지를 가져온다. |

<br />
<br />
<br />

10. 기존 매니페스트 파일 수정

`spring-pod.yaml`

```yaml
apiVersion: v1
kind: Pod

metadata:
name: spring-pod

spec:
containers:
  - name: spring-container
  image: spring-server
  ports:
    - containerPort: 8080
  imagePullPolicy: IfNotPresent

# 위와 같이 정책을 설정해주어야 로컬에서 이미지를 가져오게 된다.
```

<br />
<br />
<br />

11. 기존 파드를 삭제하고 다시 생성해보자

```
$ kubectl delete pod spring-pod

$ kubectl apply -f spring-pod.yaml

$ kubectl get pods
```

<br />
<br />
<br />

12. 서버 작동 확인

<br />

`파드 내부로 들어가서 요청보내기`

```
$ kubectl exec -it spring-pod -- bash

$ curl localhost:8080
```

<br />


`포트 포워딩 활용하기`

```
# 원하는 포트로 포트포워딩 한다.
$ kubectl port-forward pod/spring-pod 1234:8080
```

<br />
<br />
<br />

13. 파드 삭제하기

```
# 필요없는 파드를 전부 삭제하자
$ kubectl delete pod spring-pod
```
