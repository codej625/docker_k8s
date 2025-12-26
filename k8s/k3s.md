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
k3s의 권장 설치 사양은 CPU 2core, Memory 2GB 이다.

백엔드 서버도 같이 실행시킬 것을 고려하면,
AWS EC2 인스턴스는 t4g.medium로 생성하면 좋다.
```

<br />
<br />
<br />

2. k3s 설치하기

```zsh
curl -sfL https://get.k3s.io | sh - # k3s 설치
sudo chmod 644 /etc/rancher/k3s/k3s.yaml # 권한 부여
sudo kubectl version # k3s 잘 설치됐는 지 확인

# kubectl 명령어 자동 완성 추가
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc

# kubectl 설정 (zsh)
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.zshrc
source ~/.zshrc

# k3s 노드 확인
kubectl get nodes
```

<br />
<br />
<br />

3. k3s 사용해서 간단한 데이터베이스 서버 만들기 (실습)

```
해당 예시에서는 ServiceLB를 사용해서 MetalLB를 대신하고
간단한 데이터베이스 서버를 만들어볼 것이다.

* ServiceLB란?
ServiceLB는 K3s에 기본으로 내장된 간단한 LoadBalancer 구현체이다.
```

<br />

```zsh
# Longhorn 필수 패키지 한 번에 설치 (iSCSI, NFS)
sudo apt update && sudo apt install -y open-iscsi nfs-common

# iSCSI 서비스 시작 및 활성화
sudo systemctl enable --now iscsid

# 서비스 상태 확인 (active (running) 인지 확인)
sudo systemctl status iscsid

# Longhorn 설치 (Longhorn 설치 후 약 1~2분 대기 필수)
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml

# Longhorn 설치 확인
kubectl get storageclass | grep longhorn # longhorn (io.rancher.longhorn)

# multipathd 비활성화 (Longhorn과 충돌 방지)
sudo systemctl stop multipathd
sudo systemctl disable multipathd

# dm_crypt 커널 모듈 로드
sudo modprobe dm_crypt
echo "dm_crypt" | sudo tee -a /etc/modules

# Metrics Server 설치 (리소스 모니터링용 - 선택)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Metrics Server와 Kubelet 간의 TLS 인증서 검증을 건너뛰어 리소스 데이터 수집 에러를 해결함
kubectl patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

# Longhorn Storage Reserved 조정 (디스크 용량이 부족한 경우)
# 먼저 노드 이름과 디스크 ID 확인
kubectl get nodes.longhorn.io -n longhorn-system  # 노드 이름 확인
kubectl get nodes.longhorn.io <조회한-노드-이름> -n longhorn-system -o yaml  # 디스크 ID 확인

# 확인한 정보로 storageReserved 조정 (3GB로 설정)
kubectl patch nodes.longhorn.io <node-name> -n longhorn-system --type='json' \
  -p='[{"op": "replace", "path": "/spec/disks/<disk-id>/storageReserved", "value": 3221225472}]'
```

<br />

```zsh
# AWS EC2 보안 그룹 설정 (AWS 콘솔에서 먼저 진행)
# 인바운드 규칙에 TCP 5432 포트 추가 필수 (외부 접속용)
# 접속할 IP 또는 0.0.0.0/0 (전체 허용 시)

# K3s 필수 포트 허용
sudo ufw allow 22/tcp
sudo ufw allow 6443/tcp # Kubernetes API Server
sudo ufw allow 10250/tcp # Kubelet Metrics
sudo ufw allow 8472/udp # Flannel VXLAN (파드 간 네트워크 통신 - 필수)

# PostgreSQL 외부 접속 허용 (ServiceLB 사용 시)
sudo ufw allow 5432/tcp

# 방화벽 활성화
sudo ufw enable
```

<br />
<br />
<br />

4. 매니페스트 파일 만들기

<br />

`postgres-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: postgres-config
  namespace: postgres-database # 네임스페이스

data:
  # 데이터베이스 이름
  postgres-database: healthapp
```

<br />

`postgres-secret.yaml`

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: postgres-secret
  namespace: postgres-database # 네임스페이스

stringData:
  # 데이터베이스 사용자 이름
  postgres-user: "healthapp"
  # 데이터베이스 사용자 비밀번호
  postgres-password: "#1234"
```

```zsh
# 실제 서버 운영 시 명령어로 바로 적용하고 매니페스트 파일 남기지 않는 방법도 있음
kubectl create secret generic postgres-secret \
  --namespace=postgres-database \
  --from-literal=postgres-user=healthapp \
  --from-literal=postgres-password=#1234
```

<br />

`postgres-storageclass.yaml`

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1

metadata:
  name: longhorn
  # name: longhorn-single

provisioner: driver.longhorn.io
allowVolumeExpansion: true # PVC 용량 확장 허용
parameters:
  # 단일 노드 환경에서는 1로 설정 필수
  numberOfReplicas: "1" # Longhorn은 3개의 복제본을 기본으로 하지만, 현재처럼 단일 노드(EC2 1대) 환경에서는 복제본을 1로 설정해야만 볼륨이 정상적으로 작동
  staleReplicaTimeout: "2880"

# Longhorn 설치 시 기본 StorageClass가 자동 생성되므로,
# 단일 노드 환경에서는 이 파일을 사용하여 replica를 1로 고정한 별도의 StorageClass를 생성
```

<br />

`postgres-pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: postgres-pvc
  namespace: postgres-database # 네임스페이스

spec:
  storageClassName: longhorn
  # storageClassName: longhorn-single
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

<br />

`postgres-statefulset.yaml`

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: postgres-statefulset
  namespace: postgres-database # 네임스페이스

spec:
  serviceName: postgres-headless # headless service
  replicas: 1 # 스케일 아웃 (배포 대수)
  selector:
    matchLabels:
      app: postgres-db

  template:
    metadata:
      labels:
        app: postgres-db
    spec:
      containers:
        - name: postgres-container # 컨테이너 이름
          image: postgres:15 # 이미지 이름
          imagePullPolicy: IfNotPresent # 이미지 풀 정책 (Always, Never, IfNotPresent)
          ports:
            - containerPort: 5432 # 컨테이너 포트 번호
          resources:
            requests:
              memory: "256Mi"
              cpu: "50m"
            limits: # 리소스 제한 (최대)
              memory: "1408Mi" # 메모리 제한 - 사용량은 사양에 따라 변경
              cpu: "1200m" # CPU 제한 - 사용량은 사양에 따라 변경
              # memory: "1408Mi" # 메모리 4GB 할당 기준
              # cpu: "1200m" # vCPU 2코어 할당 기준
          env:
            # 환경 변수 설정
            ## 데이터베이스 설정
            ### 데이터베이스 이름
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: postgres-config
                  key: postgres-database
            ### 데이터베이스 사용자 이름
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: postgres-user
            ### 데이터베이스 사용자 비밀번호
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: postgres-password
            ### 서브디렉토리 사용
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          # 볼륨 마운트
          volumeMounts:
            - name: postgres-persistent-storage # 볼륨 이름
              mountPath: /var/lib/postgresql/data # 볼륨 마운트 경로 - PostgreSQL 권장 경로

      # 볼륨 설정
      volumes:
        - name: postgres-persistent-storage # 볼륨 이름 (PV)
          persistentVolumeClaim:
            claimName: postgres-pvc # pvc 이름
```

<br />

`headless-service.yaml`

```yaml
apiVersion: v1
kind: Service

metadata:
  name: postgres-headless
  namespace: postgres-database # 네임스페이스

spec:
  clusterIP: None # Headless Service의 핵심
  selector:
    app: postgres-db
  ports:
    - port: 5432
      targetPort: 5432
```

<br />

`postgres-service.yaml`

```yaml
apiVersion: v1
kind: Service

metadata:
  name: postgres-service
  namespace: postgres-database # 네임스페이스

spec:
  type: LoadBalancer # AWS EC2 인스턴스의 보안 그룹에서 인바운드 규칙으로 TCP 5432 포트가 열려 있지 않으면 외부(DBeaver나 백엔드 서버 등)에서 접속할 수 없음
  # type: NodePort # NodePort 설정 시, 외부 접속 사용 가능 (ClusterIP, NodePort, LoadBalancer)
  selector:
    app: postgres-db
  ports:
    - protocol: TCP
      targetPort: 5432
      port: 5432
      name: postgres
      # nodePort: 30000 # NodePort 설정 시 사용 nodePort를 명시하지 않으면, 랜덤으로 지정 (30000~)
# type가 ClusterIP 일 때 외부 접속 필요 시, kubectl port-forward pod/[pod-name] [local-port]:[container-port] 명령어 사용
```

<br />
<br />
<br />

5. 매니페스트 파일 적용

<br />

`네임스페이스 먼저 생성 (이미 있으면 무시)`

```zsh
kubectl create namespace postgres-database --dry-run=client -o yaml | kubectl apply -f -

# 순서대로 적용
kubectl apply -f postgres-config.yaml
kubectl apply -f postgres-secret.yaml

kubectl apply -f postgres-storageclass.yaml
kubectl apply -f postgres-pvc.yaml

kubectl apply -f headless-service.yaml
kubectl apply -f postgres-statefulset.yaml
kubectl apply -f postgres-service.yaml
```

<br />

```zsh
kubectl get all,configmap,secret,pvc -n postgres-database # database 네임스페이스에 다 있어야 함
kubectl get pv # PV는 클러스터 전체
```

<br />

```zsh
# PostgreSQL 실제 사용량 확인
kubectl top pod -n postgres-database

# 노드 전체 자원 상황 확인
kubectl top node
```

<br />
<br />
<br />

6. 팁

<br />

`Longhorn 대시보드 보기`

```zsh
# longhorn-system 서비스 확인
kubectl get svc -n longhorn-system

# longhorn-frontend 서비스를 port-forward로 열기
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80

# 브라우저에서 http://localhost:8080 접속 (볼륨 상태, 백업 설정 등 관리 가능)
```

<br />

`스토리지 늘리기`

```zsh
# storage: 20Gi -> 50Gi로 변경 (Longhorn이 자동 확장)
kubectl edit pvc postgres-pvc -n postgres-database
```

<br />

`디비 자동 스냅샷`

```
* 볼륨의 자동 스냅샷(snapshot)과 백업(backup)을 주기적으로 실행하도록 스케줄링하는 기능

Longhorn UI에서 Volume -> Recurring Jobs 설정하기
```

<br />

`ufw 방화벽 설정`

```zsh
# 상태 확인
sudo ufw status verbose

# 방화벽 활성화
sudo ufw enable

# 포트 및 서비스 허용/차단 규칙 (예시)
sudo ufw allow 22/tcp # 22번 포트 허용

sudo ufw allow 443/tcp # HTTPS 포트 허용

sudo ufw allow 53/udp  # DNS (UDP) 포트 허용

#특정 IP 주소에서 특정 포트 허용
sudo ufw allow from 192.168.1.100 to any port 22

# 기본 http 프로토콜(80, 443) 허용
sudo ufw allow http
sudo ufw allow https

# Telnet 포트 차단
sudo ufw deny 23/tcp

# 또는 번호로 삭제 (sudo ufw status numbered로 번호 확인)
sudo ufw delete 1
sudo ufw delete allow 80/tcp
```

<br />

`트러블슈팅`

```zsh
# Pod가 ContainerCreating 상태에서 멈춘 경우
kubectl describe pod <pod-name> -n <namespace>  # Events 섹션 확인

# 볼륨 생성 실패 시 (insufficient storage)
kubectl get nodes.longhorn.io <node-name> -n longhorn-system -o yaml
# storageReserved 값을 줄이거나 PVC 용량을 줄여서 해결

# Pod가 Error 상태인 경우
kubectl logs <pod-name> -n <namespace>  # 로그 확인
kubectl logs <pod-name> -n <namespace> --previous  # 이전 컨테이너 로그

# PostgreSQL "directory exists but is not empty" 에러
# -> PGDATA 환경 변수를 /var/lib/postgresql/data/pgdata로 설정

# Longhorn 볼륨이 faulted 상태인 경우
kubectl delete volumes.longhorn.io --all -n longhorn-system  # 볼륨 초기화
```

<br />

`PostgreSQL 초기화 실패`

```zsh
# "directory exists but is not empty" 에러 해결
# StatefulSet에 PGDATA 환경 변수 추가 필요
- name: PGDATA
  value: /var/lib/postgresql/data/pgdata
```
