# 도커 파일 만들기

<br /><br />

1. 간단하게 도커 파일을 만들어보자.
```
# 사용할 베이스 이미지를 선택. OpenJDK 11을 사용.
FROM openjdk:17-jdk-slim

# Root 디렉토리 설정
WORKDIR /app

# JAR 파일을 Docker 이미지로 복사 [원본 파일 Path] [도커 내부(Linux) Path]
COPY app.jar .

# 컨테이너가 시작될 때 실행할 명령어를 설정
CMD ["java", "-jar", "app.jar"]
```

<br />

```
예시)
이런 식으로 실행 순서를 정할 수 있다.

FROM tomcat:9.0

COPY ./ojdbc6-11.2.0.4.jar /usr/local/tomcat/lib/ojdbc6-11.2.0.4.jar

RUN echo 'CATALINA_OPTS="$CATALINA_OPTS -Doracle.jdbc.implicitStatementCacheSize=20 -Doracle.jdbc.defaultRowPrefetch=20"' > /usr/local/tomcat/bin/setenv.sh
RUN echo 'CLASSPATH=$CLASSPATH:/usr/local/tomcat/lib/ojdbc8.jar' >> /usr/local/tomcat/bin/setenv.sh

COPY ./kolas.war /usr/local/tomcat/webapps/kolas.war

CMD ["catalina.sh", "run"]
```

<br />

2. 도커 파일 빌드.
```
# -f 명령어를 사용해서 app.dockerfile [파일 경로]을 my-app이라는 이름으로 바꿈
# -t 명령어는 태그를 설정

docker build -t my-app -f app.dockerfile .
```

<br />

3. 도커 이미지 완성

<br />

4. 도커 이미지를 컨테이너화 시키기
```
docker images
-> 도커의 이미지 리스트
```
```
# run 컨테이너 생성과 실행
# -d 백그라운드 실행
# -p 포트 매칭

docker run -d -p 8080:80 {image_name}
```
