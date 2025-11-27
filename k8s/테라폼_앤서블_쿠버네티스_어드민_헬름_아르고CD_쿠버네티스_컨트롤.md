# K8s 배포까지 전체 자동화 파이프라인을 위한 기술들을 알아보자.

<br />
<br />

* 인프라를 구축하기 위한 도구들

---

```
K8s는 인프라 구축을 편하게 만들어주는 훌륭한 도구이다.

하지만, 인프라 구축을 위한 K8s의 구축을 쉽게 만들어주는 도구 또한 존재한다.

여기서는 간단히 정리한 표를 참고 해서 단계별로 사용되는 도구들을 살펴보자.
```


<br />
<br />
<br />
<br />

1. 인스턴스 생성부터 K8s 배포까지의 전체 자동화 파이프라인

```
이 표는 클라우드 환경에서 자체 관리형 쿠버네티스(Self-Managed Kubernetes) 클러스터를 구축하고, 
애플리케이션을 배포하는 데 사용되는 주요 도구와 그 역할을 요약했다.
```

<br />

| 단계 | 주요 도구 | 관리 대상 | 핵심 작업 | 예시 (코드/작업 내용) |
| :--- | :--- | :--- | :--- | :--- |
| **1. 인프라 프로비저닝 (IaC)** | **Terraform** | 클라우드 인프라 (VPC, VM, 스토리지) | VM 인스턴스(Master/Worker) 및 네트워크 환경을 코드로 생성하고 관리 | `aws_instance` 리소스 정의, `subnet` 및 `security_group` 설정 |
| **2. OS 및 패키지 설치** | **Ansible** | VM (Host OS) | 생성된 VM에 SSH로 접속하여 **Docker/Containerd** 설치, **`kubeadm`** 등 쿠버네티스 필수 패키지 설치, Host OS 설정(Swap 비활성화 등)을 자동화 | `apt` 모듈로 Docker 설치, `sysctl` 모듈로 커널 파라미터 설정 |
| **3. 클러스터 초기화** | **Kubeadm** | 쿠버네티스 클러스터 | 마스터 노드 초기화(`kubeadm init`) 및 워커 노드 조인(`kubeadm join`) 명령을 실행하여 VM들을 하나의 클러스터로 구성 | 마스터 노드에서 `$ kubeadm init ...` 실행, 토큰을 이용한 워커 노드 조인 |
| **4. 애플리케이션 패키징** | **Helm** | 쿠버네티스 매니페스트 | 애플리케이션 배포에 필요한 모든 YAML 파일(Deployment, Service 등)을 **Chart** 형태로 템플릿화하고 패키징 | `values.yaml` 파일에 이미지 태그와 복제본 수 정의, `templates/deployment.yaml` 작성 |
| **5. 지속적인 배포 (CD)** | **ArgoCD / Flux** | Git 저장소 & 클러스터 | GitOps 방식으로 Git에 정의된 **Helm Chart**를 읽어 클러스터에 자동으로 적용(**`apply`** 역할)하고, 클러스터의 상태를 지속적으로 동기화 | ArgoCD에 Git 리포지토리 및 Helm Chart 경로 등록, Auto-sync 활성화 |
| **6. 런타임 상호작용** | **Kubectl** | 쿠버네티스 API | 클러스터 상태 확인, 수동으로 리소스 적용/삭제 등 클러스터와 직접 소통하는 데 사용 | `$ kubectl get pods`, `$ kubectl apply -f my-app.yaml` |
