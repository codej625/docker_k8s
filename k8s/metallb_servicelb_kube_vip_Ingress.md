# K3s Ingress 진입 방법 정리

<br />
<br />

* 베어메탈(VM) 환경에서 Ingress 설정 시, 어떤 접근 방법이 좋을까?

---
Service는 포트를 받는 것이다.

노드(워커)는 그 Service까지 패킷이 도달하는 경로라고 생각하면 편하다.

외부에서 접속 시, 노드로 접근하는 가장 좋은 방법을 찾아보자.
---

<br />
<br />
<br />
<br />

1. 트래픽 흐름 (순서)

| 단계 | 설명 |
|:---:|-----|
| 1 | 도메인 접속 (클라이언트) → DNS가 공인 IP로 변환 |
| 2 | 네트워크 장비 포트포워딩: 외부 IP + 80/443 → 내부 IP + 80/443 |
| 3 | 내부 IP로 들어온 패킷 → 그 IP를 "가지고 있는" 노드로 전달 |
| 4 | 그 노드의 kube-proxy가 해당 포트를 Ingress Controller **Service**로 넘김 |
| 5 | Service → Ingress Controller **Pod** (Traefik 또는 Nginx) |
| 6 | Ingress Pod가 host/path에 따라 해당 앱 **Service**로 전달 |
| 7 | 앱 Service → 앱 **Pod**가 처리·응답 |

<br />
<br />

`3단계에서 "IP를 가지고 있는 노드"란?`

| 종류 | 설명 |
|:---:|-----|
| MetalLB | 한 노드의 Speaker가 풀 IP를 ARP로 보유. 그 노드가 죽으면 다른 워커 Speaker가 이어받음. |
| Klipper | 도메인/포트포워딩에 적어 둔 그 워커 노드의 실제 IP. |

<br />
<br />
<br />

2. Klipper(ServiceLB) vs MetalLB vs kube-vip

| | Klipper | MetalLB | kube-vip |
|--|--------|--------|----------|
| **가상 IP** | 없음 | IP 풀에서 서비스마다 하나씩 할당 | 가상 IP 한두 개만 정해 둠 (풀 아님) |
| **Service에 붙는 주소** | 노드 실제 IP 중 하나를 우리가 선택 | 풀에서 나온 IP 하나 (고정 가능) | 미리 정한 가상 IP 하나(또는 소수) |
| **노드가 죽으면** | 그 노드 IP로 오는 요청 끊김 | 풀 IP가 다른 노드로 넘어가 접속 유지 | 가상 IP가 다른 노드로 넘어가 접속 유지 |
| **구성** | K3s 기본, 별도 설치 없음 | Controller + Speaker(DaemonSet) | DaemonSet만 (Controller 없음) |
| **쓰기 좋은 경우** | 단일 노드 또는 노드 IP 직접 사용 | LoadBalancer 서비스가 여러 개일 때 | 가상 IP 한두 개면 충분할 때 (예: Ingress 하나만) |

<br />

| 종류 | 설명 |
|:---:|-----|
| Klipper | 가상 IP 없음 → 노드 IP 하나로 고정 → 그 노드 죽으면 장애. |
| MetalLB | IP 풀(예: .240~.250)을 두고 서비스마다 풀에서 IP 할당. |
| kube-vip | 가상 IP 한두 개만 쓰는 방식. Traefik 하나만 쓸 때 적합. |

<br />
<br />
<br />

3. 환경별 선택

| 환경 | 추천 | 비고 |
|------|------|------|
| **노드 1대** (저사양 EC2 한 대) | K3s 기본 ServiceLB(Klipper) | MetalLB/kube-vip 불필요, 리소스 절약 |
| **EC2 3대 (AWS)** | AWS Load Balancer Controller → ALB/NLB | MetalLB는 AWS L2/ARP 제한으로 비추 |
| **VM 3대** (온프레미/비-AWS) | Ingress 하나면 kube-vip / 서비스 여러 개면 MetalLB | |

<br />
<br />
<br />

4. VM 환경에서 성능·리소스 (MetalLB vs kube-vip)

| 항목 | MetalLB | kube-vip |
|------|--------|----------|
| **구성** | Controller 1개 + Speaker(노드당 1파드) | DaemonSet만 (노드당 1파드) |
| **파드 수** | 노드 3개 시 Controller 1 + Speaker 3 = 4개 | 노드 3개 시 3개 |
| **리소스** | Controller로 풀/할당 처리 → 상대적으로 더 사용 | DaemonSet만 → 더 적게 사용 |
| **설정** | 풀/Advertisement 등 설정 많음 | 매니페스트 한두 개, VIP만 지정 |

<br />

```
처리량/지연은 둘 다 L2 ARP 위주라 비슷.

병목은 보통 Traefik/앱 쪽.

VM 기준 리소스·설정은 kube-vip가 더 가벼움.
```

<br />
<br />
<br />

5. 프로덕션에서 IP 고정 (MetalLB 사용 시)

```
Ingress Controller용 LoadBalancer Service에
풀 안에 IP 중 하나를 고정한다.
```

```yaml
metadata:
  annotations:
    metallb.universe.tf/loadBalancerIPs: "192.168.1.240"
spec:
  type: LoadBalancer

# 예시 -> 포트포워딩은 .240:80, .240:443 으로 고정. 도메인은 공인 IP(고정) → 공유기에서 .240으로 포워딩.  
```

<br />
<br />
<br />

6. 구조 요약

```
+-----------------------------------------+
|   외부 (클라이언트)                        |
|   도메인 → DNS → 공인 IP                  |
+-----------------------------------------+
                    |
                    | 포트포워딩 (외부 80/443 → 내부 IP:80/443)
                    v
+-----------------------------------------+
|   내부 네트워크                            |
|   "그 IP를 가진 노드" (MetalLB Speaker 또는 Klipper용 워커) |
+-----------------------------------------+
                    |
                    | kube-proxy
                    v
+-----------------------------------------+
|   Ingress Controller Service             |
|   (포트 80/443 수신)                       |
+-----------------------------------------+
                    |
                    v
+-----------------------------------------+
|   Ingress Controller Pod                 |
|   (Traefik / Nginx)                      |
+-----------------------------------------+
                    |
                    | host/path 라우팅
                    v
+-----------------------------------------+
|   앱 Service → 앱 Pod                     |
+-----------------------------------------+
```
