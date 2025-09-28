# 도커의 기본 개념을 알아보자!

<br /><br />

* What is docker?
---

```
Container vs VM

VM은 가상의 머신을 만드는 것인데,
가상 머신마다 운영체제를 포함하기 때문에 매우 무겁다.

반면, 도커는 Container Engine(Doker Engine)이 설치된
호스트 운영체제를 공유하며 Container 단위로 실행이 된다.

운영체제가 포함되지 않기 때문에 VM 단위 작동보다
Container 단위 작동이 훨씬 적은 리소스로 가볍게 작동한다.
```

<br />

| 항목                | 도커(Container)                                                                                                                                         | 가상 머신(Virtual Machine)                                                                                                                                                |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **격리 수준**       | 프로세스 격리 (호스트 OS의 커널을 공유)                                                                                                                  | 하드웨어 수준의 격리 (호스트 OS 위에 게스트 OS 실행)                                                                                                                     |
| **운영 체제**       | 호스트 OS의 커널을 공유하며, 각 컨테이너는 독립된 응용 프로그램과 라이브러리를 포함합니다.                                                                       | 각 VM은 자체 게스트 OS를 실행하며, 하이퍼바이저가 이를 관리합니다.                                                                                                       |
| **부팅 시간**       | 매우 빠름 (몇 초 내외)                                                                                                                                     | 비교적 느림 (몇 분 소요)                                                                                                                                                 |
| **자원 효율성**     | 더 가볍고 자원을 효율적으로 사용 (호스트 OS의 커널을 공유하므로 중복된 자원이 적음)                                                                            | 더 무겁고 자원 소모가 큼 (각 VM이 별도의 게스트 OS를 실행하므로 중복된 자원이 많음)                                                                                       |
| **배포 단위**       | 컨테이너 이미지 (애플리케이션과 필요한 모든 파일을 포함)                                                                                                   | VM 이미지 (전체 운영 체제와 애플리케이션을 포함)                                                                                                                         |
| **종속성 관리**     | 각 컨테이너는 독립적으로 애플리케이션과 라이브러리를 포함하므로 종속성 문제를 피할 수 있음                                                                         | 각 VM은 독립된 운영 체제를 가지므로 종속성 문제를 효과적으로 격리할 수 있음                                                                                               |
| **운영 체제 지원**  | 호스트 OS와 같은 커널을 사용해야 함 (예: 리눅스 커널을 사용하는 도커는 리눅스 기반 컨테이너만 실행 가능)                                                        | 다양한 운영 체제를 지원 (예: 호스트가 리눅스일 때 VM은 윈도우, 리눅스, 맥OS 등을 실행할 수 있음)                                                                         |
| **사용 사례**       | 마이크로서비스, CI/CD 파이프라인, 애플리케이션 격리, 환경 일관성 유지 등                                                                                     | 완전한 OS 환경이 필요한 경우, 다양한 OS를 테스트해야 하는 경우, 기존 VM 기반 인프라를 사용하는 경우                                                                        |

<br /><br /><br />

* 발전 배경
---

```

Traditional Deployment -> Virtualized Deployment -> Container Deployment 순서로 발전

```

| Traditional Deployment | Virtualized Deployment | Container Deployment |
|------------------------|------------------------|----------------------|
| X                      | ┌ App                  | ┌ App                |
| X                      | ├ Bin / Lib            | ├ Bin / Lib          |
| X                      | ├ Operating System     | ├ X                  |
| App                    | └ Virtual Machine      | └ Container          |
| Bin / Lib              | Hypervisor             | Container Runtime    |
| Operating System       | Operating System       | Operating System     |
| Hardware               | Hardware               | Hardware             |

<br />

1. Traditional Deployment
```
Hardware에 Operating system(OS)이 올라가 있고,
그 위로 App가 설치되어있다.
서버의 자원이 남아도 효율적으로 활용하기 힘든 구조이다.
```

<br />

2. Virtualized Deployment
```
Operating system(OS)에 가상화 기술을 사용한다.
VM(Virtual Machine) 단위로 가상의 서버를 구현하는 구조이다.
VM으로 완벽히 영역을 나눌 수 있지만,
VM마다 Operating system이 필요하다.
```

<br />

3. Container Deployment
```
Container 단위로 서버를 관리하고,
Operating system을 공유한다.
Docker와 같은 Container runtime을 사용한다.

마지막으로 가장 큰 차이점은 위에 표에는 존재하지 않지만,
컨테이너화된 애플리케이션의 배포, 확장 및 관리를 자동화하는
Container orchestration engine(Kubernetes)이 존재한다는 것이다.
```

<br /><br /><br />

* Docker 핵심 구성요소
---

1. 도커 파일(Dockerfile)

```
도커 이미지를 빌드하기 위한 설정 파일이다.
컨테이너가 어떻게 구성되어야 하는지에 대한 명령어와 설정이 포함된다.
도커 파일을 빌드하면 도커 이미지가 된다.

ex) Copy files, Install dependencies, Assets..등
```

<br />

2. 이미지(Image)
```
도커 컨테이너를 실행하기 위한 파일과 설정 등을 포함한 가볍고 독립적인 패키지이다.
도커 허브와 같은 저장소에서 공유할 수 있다.

ex) 자바로 치면 Class와 비슷하다.
    한번 만들어진 image는 불변 상태로 존재한다.
```

<br />

3. 컨테이너(Container)
```
이미지를 기반으로 실행된 가상 환경이다.
컨테이너는 격리된 환경에서 동작하며,
호스트 시스템과는 독립적으로 실행된다.

일반적으로 하나 혹은 여러개의 이미지 레이어를 합쳐서
하나의 컨테이너 레이어를 만든다.

여기서 이미지는 불변속성이기 때문에 Read only이고,
컨테이너의 설정은 변경이 가능하다.

ex) 자바로 치면 Class(Image)를 Instance 화 한 것과 비슷하다.
실제로 작동중인 컨테이너를 Running instance 상태라고 한다.
```

<br />

4. 도커 허브(Docker Hub)
```
도커 이미지를 저장하고 공유하는 온라인 저장소이다.
다양한 공개 이미지를 찾고, 자신의 이미지를 업로드하거나 다운로드할 수 있다.
```

<br /><br />

```
도커는 Git과 Github의 개념과 매우 유사하다.
사용 순서는 아래와 같다.

1) Local에서 Doker를 설치한다.

2) Dokerfile을 Build하여 Image를 만든다.

3) 만들어진 Image를 Container Registry(Doker hub)에 Push 한다.

4) 사용하려는 서버에 Doker를 설치 후 Image를 Pull 받는다.
```

<br /><br /><br />

* Containerizing
---

```
컨테이너화를 해서 가장 이득인 부분이 뭘까?
바로 Deployment이다.

이전에는 개발자들에게 Code와 Dependencies를
세팅하는 메뉴얼을 전달받고,
DevOps 혹은 SRE들이 스크립트, 코드를 작성해서 배포를 했다.

하지만 중간에 문제가 생기면,
개발자들과 함께 문제를 해결하는데 많은 시간이 필요했다.

지금은 개발자들에게 Docker image만 전달받으면 되기 때문에
Deploy가 굉장히 쉬워지고 빨라졌다.
```
