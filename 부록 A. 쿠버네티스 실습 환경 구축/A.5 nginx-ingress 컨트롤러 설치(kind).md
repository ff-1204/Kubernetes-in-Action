# A.5 nginx-ingress 컨트롤러 설치(kind)

> 책의 Ingress 예제를 그대로 따라가기 위한 **nginx-ingress 컨트롤러** 설치 가이드. kind는 Ingress 컨트롤러를 기본 포함하지 않으므로 별도 설치가 필요합니다.

## 사전 요구사항

- [A.2 쿠버네티스 클러스터 설치(kind)](A.2%20쿠버네티스%20클러스터%20설치(kind).md) 까지 완료된 상태
- 호스트(Windows) 의 80, 443 포트가 비어있음
- WSL2 + Docker Engine + kind + kubectl 동작

> Ingress 챕터에 도달하기 전이라면 이 문서는 미뤄도 됩니다. 책에서 Ingress가 처음 등장할 때 돌아오세요.

---

## Ingress가 뭐고 왜 따로 깔아야 하나?

**Ingress** = 클러스터 외부에서 들어오는 HTTP(S) 트래픽을 내부 `Service`로 라우팅해주는 "대문". 도메인·경로 기반 라우팅, TLS 종료, 가상 호스팅 등을 담당합니다.

쿠버네티스 표준에는 Ingress **리소스(YAML 스펙)** 만 정의돼 있고, **실제로 트래픽을 처리하는 컨트롤러(파드)** 는 사용자가 직접 설치해야 합니다. kind는 컨트롤러를 기본 포함하지 않으므로, 책의 Ingress 예제(`nginx-ingress` 기준)를 그대로 따라가려면 이 문서의 단계를 거쳐야 합니다.

```
[외부 요청]
    ↓
[Ingress Controller (nginx)]   ← 실제 트래픽을 받는 파드
    ↓ (Ingress 리소스의 규칙대로 라우팅)
[Service A] [Service B] [Service C]
    ↓
[Pod] [Pod] [Pod]
```

---

## 1. 사전 점검: 호스트 포트 80/443

> ⚠️ **이 절을 실행하면 A.2에서 만든 클러스터(`kind-ysmoon`)가 삭제되고 새로 만들어집니다.** 클러스터 안에 띄워둔 Pod·Deployment·Service 등은 모두 사라집니다. 보존할 매니페스트가 있다면 먼저 export하세요.
>
> ```bash
> kubectl get all -A -o yaml > backup.yaml
> ```

호스트의 80/443 포트가 비어있는지 확인합니다(다른 프로세스가 점유 중이면 클러스터 생성이 실패합니다). PowerShell에서:

```powershell
Get-NetTCPConnection -LocalPort 80,443 -State Listen -ErrorAction SilentlyContinue
```

빈 결과면 OK. 점유 중이라면 해당 프로세스(IIS, Skype, 기존 도커 컨테이너 등)를 먼저 정리합니다.

---

## 2. Ingress용 클러스터 재생성

기존 클러스터를 삭제하고 80/443 포트 매핑이 추가된 클러스터를 다시 만듭니다.

```bash
kind delete cluster --name ysmoon

cat <<EOF | kind create cluster --name ysmoon --image kindest/node:v1.35.0 --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

**YAML 옵션 의미:**

| 항목 | 역할 |
|------|------|
| `node-labels: "ingress-ready=true"` | kind용 nginx-ingress 매니페스트는 **이 라벨이 붙은 노드에만 컨트롤러를 스케줄링**하도록 설계됨. 라벨이 없으면 컨트롤러 파드가 `Pending` 상태로 멈춥니다 |
| `extraPortMappings 80/443` | 컨테이너로 동작하는 kind 노드의 80/443 포트를 **호스트(Windows)** 의 80/443에 매핑. 이게 있어야 브라우저에서 `http://localhost`로 접근 가능 |
| `--image kindest/node:v1.35.0` | A.2와 동일한 Kubernetes 버전 유지 (생략 시 kind 기본 버전이 사용됨) |

---

## 3. nginx-ingress 컨트롤러 설치

노드가 Ready 상태가 될 때까지 기다린 뒤, **kind 전용** nginx-ingress 매니페스트를 적용하고 controller 파드가 ready되기를 기다립니다.

```bash
kubectl wait --for=condition=Ready --timeout=120s nodes --all

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=180s
```

> 일반 nginx-ingress 매니페스트가 아니라 **`provider/kind/deploy.yaml`** 을 써야 합니다. kind 환경의 `hostPort` + `nodeSelector: ingress-ready=true` 에 맞춰진 변형 매니페스트입니다.
> 첫 이미지 풀 시 controller가 ready되기까지 1~2분 걸릴 수 있어 `--timeout=180s`로 여유를 둡니다.

---

## 4. 설치 검증

```bash
kubectl get pods -n ingress-nginx
kubectl get ingressclass
```

**출력 예:**

```text
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-xxxxx       0/1     Completed   0          1m
ingress-nginx-admission-patch-xxxxx        0/1     Completed   0          1m
ingress-nginx-controller-xxxxxxxxx-xxxxx   1/1     Running     0          1m

NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       1m
```

- `*-admission-*` 파드는 **Job**(웹훅 인증서 생성용 일회성 작업)이라 종료 후 `Completed`가 정상 — 에러 아닙니다.
- `ingress-nginx-controller`가 `1/1 Running` + IngressClass `nginx` 등록 = 성공.

---

## 5. 동작 확인 (mini-test)

실제로 호스트 → 클러스터 → 파드 경로가 뚫렸는지 간단한 nginx 파드로 시험해봅니다.

```bash
kubectl create deployment hello --image=nginx --port=80
kubectl expose deployment hello --port=80

cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 80
EOF
```

호스트(Windows)나 WSL에서 80포트로 접근:

```bash
curl http://localhost
```

nginx 기본 페이지(`<h1>Welcome to nginx!</h1>` 포함)가 반환되면 전체 경로
**호스트:80 → kind 노드 → ingress-nginx → Service → Pod** 가 정상 동작하는 것입니다.

테스트가 끝났으면 리소스를 정리합니다.

```bash
kubectl delete ingress hello
kubectl delete service hello
kubectl delete deployment hello
```

---

## 6. 트러블슈팅

| 증상 | 원인 / 해결 |
|------|-------------|
| `kind create cluster`가 `address already in use`로 실패 | 80/443 포트를 다른 프로세스가 점유 중. 1절의 명령으로 찾아 종료 후 재시도 |
| `ingress-nginx-controller` 파드가 `Pending` | 노드에 `ingress-ready=true` 라벨이 없음. `kubectl get nodes --show-labels` 로 확인 → 누락이면 클러스터를 2절 YAML로 재생성 |
| `curl http://localhost` 가 `connection refused` | `extraPortMappings` 누락 또는 컨테이너 재시작으로 매핑 해제. `docker ps` 로 PORTS 컬럼에 `0.0.0.0:80->80` 이 있는지 확인 |
| 컨트롤러 파드가 `CrashLoopBackOff` | 이미지 풀 실패 또는 admission webhook 인증서 미생성. `kubectl logs -n ingress-nginx <pod>` 로 원인 확인 |
| Ingress 적용했는데 404 반환 | `ingressClassName: nginx` 누락 가능성. `kubectl describe ingress <이름>` 로 `Class` 필드 확인 |

> 이미 이 절을 진행했다면 80/443 매핑은 일반 학습에 영향을 주지 않으므로 그대로 사용해도 무방합니다.
