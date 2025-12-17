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
AWS EC2 인스턴스는 t4g.small로 생성하면 좋다.
```

<br />
<br />
<br />

2. Docker 설치

```zsh
# 기존 오래된 Docker 패키지 제거 (있을 경우)
sudo apt-get remove -y docker docker-engine docker.io containerd runc docker-compose docker-compose-v2 docker-doc podman-docker

# 필요 패키지 설치
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# Docker 공식 GPG 키와 리포지토리 추가
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 리포지토리 추가 (Ubuntu 버전에 자동 맞춤)
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 패키지 업데이트 후 Docker 설치 (Engine + Compose 플러그인 포함)
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 현재 사용자(예: ubuntu)를 docker 그룹에 추가 (sudo 없이 사용 가능)
sudo usermod -aG docker $USER

# 새 그룹 적용 (로그아웃 없이 즉시 적용)
newgrp docker
```

<br />
<br />
<br />

3. 설치 확인

```zsh
# 설치 확인
docker version                  # Docker Engine 버전 확인
docker compose version         # Docker Compose 버전 확인 (v5.x)
sudo docker run hello-world    # 테스트 컨테이너 실행
```

<br />
<br />
<br />

4. k3s 설치하기

```zsh
$ curl -sfL https://get.k3s.io | sh - # k3s 설치
$ sudo chmod 644 /etc/rancher/k3s/k3s.yaml # 권한 부여
$ sudo kubectl version # k3s 잘 설치됐는 지 확인
```

<br />
<br />
<br />

5. k3s 사용해서 간단한 데이터베이스 서버 만들기 (실습)

```
해당 예시에서는 ServiceLB를 사용해서 MetalLB를 대신하고
간단한 데이터베이스 서버를 만들어볼 것이다.

* ServiceLB란?
ServiceLB는 K3s에 기본으로 내장된 간단한 LoadBalancer 구현체이다.
```

<br />

```zsh
# K3s 설치
curl -sfL https://get.k3s.io | sh -

# kubectl 설정 (zsh)
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.zshrc
source ~/.zshrc

# k3s 노드 확인
kubectl get nodes

# Longhorn 설치
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml

# Longhorn 설치 확인
kubectl get storageclass | grep longhorn # longhorn   (io.rancher.longhorn)
```

<br />
<br />
<br />

6. 매니페스트 파일 만들기

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
  postgres-password: "#1234" # echo -n <비밀번호> | base64 이런 식으로 인코딩해서 넣는 것도 좋음
```

```zsh
# 실제 서버 운영 시 명령어로 바로 적용하고 매니페스트 파일 남기지 않는 방법도 있음
kubectl create secret generic postgres-secret \
  --namespace=postgres-database \
  --from-literal=postgres-user=healthapp \
  --from-literal=postgres-password=#1234
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
  name: postgres-deployment
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
            - name: POSTGRES_DATABASE
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
  type: LoadBalancer
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

7. 매니페스트 파일 적용

<br />

`네임스페이스 먼저 생성 (이미 있으면 무시)`

```zsh
kubectl create namespace postgres-database --dry-run=client -o yaml | kubectl apply -f -

# 순서대로 적용
kubectl apply -f postgres-config.yaml
kubectl apply -f postgres-secret.yaml

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

8. 팁

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
