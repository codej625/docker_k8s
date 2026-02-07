# LoadBalancer(MetalLB) vs NodePort

<br />
<br />

* Worker Node 장애 시 포트포워딩 이슈

---

```
Worker Node가 2개 있는 환경에서 Worker Node 1이 다운되면,
Master Node는 자동으로 Worker Node 2로 복구 작업을 수행한다.

이때 Worker Node 2는 다른 IP 주소를 가지고 있기 때문에,
기존에 설정한 포트포워딩 규칙을 수동으로 변경해야 하는 문제가 발생한다.
```

<br />
<br />
<br />
<br />

1. nodeport 사용 시 문제점

```zsh
# 초기 상태
Master-1 (192.168.0.10) - Master 역할
Worker-1 (192.168.0.20) - Pod 실행 중
Worker-2 (192.168.0.30) - 대기

# 공유기 포트포워딩
공인IP:80 -> 192.168.0.20:30080 # 노드포트
```

```zsh
Worker-1 # 다운 발생
    ↓
Master # Worker-2로 Pod 재배치
    ↓
Worker-2 # 192.168.0.30 에서 Pod 실행
    ↓
문제 발생 # 공유기 포트포워딩은 여전히 192.168.0.20 (Worker-1)
    ↓
사용자 요청이 죽은 Worker-1(20)로 계속 감
    ↓
수동으로 포트포워딩 변경 필요
# 공인IP:80 -> 192.168.0.30:30080 (Worker-2)
```

<br />
<br />
<br />

2. MetalLB 사용으로 문제 해결

```zsh
# 초기 상태
Master-1 (192.168.0.10) - Master 역할
Worker-1 (192.168.0.20) - Pod 실행 중
Worker-2 (192.168.0.30) - 대기

# 공유기 포트포워딩
공인IP:80 -> 192.168.0.240:80 (VIP)
MetalLB가 처음으로 요청을 받고
지정한 IP 하나를 사용 (여기서는 240)
VIP(240) -> Worker-1(20)로 라우팅

Worker-1 # 다운 발생
    ↓
Master # Worker-2로 Pod 재배치
    ↓
MetalLB # VIP(240) → Worker-2(30)로 라우팅 변경
    ↓
공유기 포트포워딩은 그대로 (240)
    ↓
사용자 요청이 자동으로 Worker-2(30)로 감
    ↓
수동 변경 불필요
```
