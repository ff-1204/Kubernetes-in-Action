# A.3 멀티노드 클러스터 구성(kind)

[A.2 쿠버네티스 클러스터 설치(kind)](A.2%20쿠버네티스%20클러스터%20설치(kind).md) 까지 완료된 상태에서 시작합니다. kind는 단일 도구로 단일 노드부터 멀티노드, 다중 클러스터, HA 시뮬레이션까지 모두 지원하므로 별도 도구 추가가 필요 없습니다.

## 1. WSL2 자원 조정 (권장)

### 메모리 한도 조정

호스트 환경별 권장값입니다(WSL2가 호스트 자원을 너무 많이 가져가면 호스트 응답성이 저하되므로 보수적으로).

| 호스트 RAM | memory | processors | swap | 적합한 사용 |
| :--- | :--- | :--- | :--- | :--- |
| 8GB | 4GB | 2 | 2GB | kind 단일 노드 |
| 16GB | 8GB | 4 | 4GB | 멀티노드 2~3개 |
| 32GB 이상 | 12~16GB | 4~6 | 4GB | HA 시뮬레이션(마스터 3 + 워커 2) 여유 |

```powershell
PS C:\windows\system32> notepad $env:USERPROFILE\.wslconfig
```

```ini
[wsl2]
memory=12GB
processors=4
swap=4GB
```

설정을 적용하려면 WSL을 한 번 재시작합니다. 다음에 `wsl` 명령으로 Ubuntu에 진입할 때 새 `.wslconfig`가 자동 적용됩니다.

```powershell
wsl --shutdown
```

### .wslconfig 적용 확인

WSL 재진입 직후 `free -h`로 위에서 작성한 `.wslconfig`(memory / swap)가 반영됐는지 확인합니다.

```bash
free -h
```

출력 예(호스트 32GB, `.wslconfig`에 `memory=12GB / swap=4GB` 적용):

```text
               total        used        free      shared  buff/cache   available
Mem:            11Gi       517Mi        10Gi       3.5Mi       488Mi        11Gi
Swap:          4.0Gi          0B       4.0Gi
```

`Mem`의 `total`이 `.wslconfig`의 `memory` 값과 비슷하고(시스템 예약분으로 약간 작게 표시), `Swap`이 `swap` 값과 일치하면 정상입니다.

## 2. kind의 멀티노드 구조

각 노드가 Docker 컨테이너로 동작합니다. 한 호스트에서 여러 노드와 클러스터를 자유롭게 띄울 수 있습니다.

```
WSL2 Ubuntu
└─ Docker Engine
    └─ kind
        └─ 클러스터 (각 노드 = Docker 컨테이너)
            ├─ control-plane 컨테이너
            ├─ worker 컨테이너 1
            ├─ worker 컨테이너 2
            └─ ...
```

> **HA 한계**: 모든 노드가 같은 호스트의 Docker 데몬에 의존하므로 **진짜 HA는 아닙니다.** 학습·CI 용도로 합의 동작과 재스케줄링을 검증하는 데 적합합니다.

## 3. 사전 점검

A.2에서 설치한 Docker, kind, kubectl이 정상 동작하는지 확인합니다.

```bash
ysmoon@YSMOON:~$ docker version
Server: Docker Engine - Community
 Engine:
  Version:          29.x.x
```

```bash
ysmoon@YSMOON:~$ kind version
kind v0.31.0 go1.25.5 linux/amd64
```

```bash
ysmoon@YSMOON:~$ kubectl version --client
Client Version: v1.36.0
```

A.2에서 만든 단일 노드 클러스터가 남아 있다면 정리합니다.

```bash
kind get clusters
# ysmoon

kind delete cluster --name ysmoon
```

## 4. 멀티노드 단일 클러스터 (control-plane 1 + worker 2)

config 파일을 작성합니다.

```bash
cat > kind-multi.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```

클러스터 생성:

```bash
kind create cluster --name multi --config kind-multi.yaml --image kindest/node:v1.35.0
```

**생성 로그**

```bash
ysmoon@YSMOON:~$ kind create cluster --name multi --config kind-multi.yaml --image kindest/node:v1.35.0
Creating cluster "multi" ...
 ✓ Ensuring node image (kindest/node:v1.35.0) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-multi"
You can now use your cluster with:

kubectl cluster-info --context kind-multi
```

확인:

```bash
ysmoon@YSMOON:~$ kubectl get nodes
NAME                  STATUS   ROLES           AGE   VERSION
multi-control-plane   Ready    control-plane   2m    v1.35.0
multi-worker          Ready    <none>          90s   v1.35.0
multi-worker2         Ready    <none>          90s   v1.35.0
```

## 5. HA 시뮬레이션 (control-plane 3 + worker 2)

운영급 HA의 etcd Raft 합의를 직접 검증할 수 있는 구성입니다. 이론적 배경은 [A.4 HA 구성](A.4%20쿠버네티스%20최소%20이중화(HA)%20구성.md) 을 참고하세요.

config 파일을 작성합니다.

```bash
cat > kind-ha.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
EOF
```

클러스터 생성:

```bash
kind create cluster --name ha --config kind-ha.yaml --image kindest/node:v1.35.0
```

**생성 로그**

```bash
ysmoon@YSMOON:~$ kind create cluster --name ha --config kind-ha.yaml --image kindest/node:v1.35.0
Creating cluster "ha" ...
 ✓ Ensuring node image (kindest/node:v1.35.0) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦 📦
 ✓ Configuring the external load balancer ⚖️
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining more control-plane nodes 🎮
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-ha"
You can now use your cluster with:

kubectl cluster-info --context kind-ha
```

> 💡 control-plane이 2개 이상이면 kind가 **외부 로드밸런서 컨테이너**(`ha-external-load-balancer`)를 자동으로 띄워 API 서버 앞단에 둡니다. 위 로그의 `Configuring the external load balancer ⚖️` 단계가 그것이며, 단일 마스터 클러스터(4절)에는 나타나지 않습니다.

확인:

```bash
ysmoon@YSMOON:~$ kubectl get nodes --context kind-ha
NAME                STATUS   ROLES           AGE   VERSION
ha-control-plane    Ready    control-plane   3m    v1.35.0
ha-control-plane2   Ready    control-plane   2m    v1.35.0
ha-control-plane3   Ready    control-plane   2m    v1.35.0
ha-worker           Ready    <none>          1m    v1.35.0
ha-worker2          Ready    <none>          1m    v1.35.0
```

### 합의 동작 검증

마스터 1대를 죽여보면서 quorum 유지를 확인합니다.

```bash
ysmoon@YSMOON:~$ docker stop ha-control-plane2
ha-control-plane2

ysmoon@YSMOON:~$ kubectl get nodes --context kind-ha
NAME                STATUS     ROLES           AGE   VERSION
ha-control-plane    Ready      control-plane   5m    v1.35.0
ha-control-plane2   NotReady   control-plane   4m    v1.35.0
ha-control-plane3   Ready      control-plane   4m    v1.35.0
ha-worker           Ready      <none>          3m    v1.35.0
ha-worker2          Ready      <none>          3m    v1.35.0
```

`ha-control-plane2`가 `NotReady`인 상태에서도 다른 명령은 정상 동작합니다(quorum: 3대 중 2대 살아있음).

```bash
ysmoon@YSMOON:~$ kubectl run nginx --image=nginx --context kind-ha
pod/nginx created
```

복구:

```bash
ysmoon@YSMOON:~$ docker start ha-control-plane2
ha-control-plane2
```

이제 한 대 더 죽여 quorum이 깨지는 경우(2대 다운)를 검증해 볼 수도 있습니다.

```bash
docker stop ha-control-plane2 ha-control-plane3
kubectl get nodes --context kind-ha --request-timeout=10s    # Unable to connect... (quorum 깨져 응답 불가)
docker start ha-control-plane2 ha-control-plane3
```

> ⚠️ `--request-timeout` 없이 실행하면 클라이언트가 **수십 초 hang**될 수 있습니다. quorum이 깨지면 API 서버가 요청을 처리하지 못해 응답이 오지 않기 때문입니다.

## 6. 다중 독립 클러스터 (dev / staging 분리)

서로 다른 환경을 동시에 운용하고 싶을 때 유용합니다.

```bash
kind create cluster --name dev --image kindest/node:v1.35.0
kind create cluster --name staging --image kindest/node:v1.35.0
```

확인:

```bash
ysmoon@YSMOON:~$ kind get clusters
dev
ha
multi
staging
```

각 클러스터는 별도의 Docker 네트워크와 노드 컨테이너를 가지므로 완전히 격리됩니다.

## 7. 컨텍스트 전환

kind는 클러스터 생성 시 `kind-<name>` 형식의 컨텍스트를 `~/.kube/config`에 자동 추가하고 현재 컨텍스트로 전환합니다.

```bash
ysmoon@YSMOON:~$ kubectl config get-contexts
CURRENT   NAME           CLUSTER        AUTHINFO       NAMESPACE
*         kind-staging   kind-staging   kind-staging
          kind-dev       kind-dev       kind-dev
          kind-ha        kind-ha        kind-ha
          kind-multi     kind-multi     kind-multi
```

전환:

```bash
ysmoon@YSMOON:~$ kubectl config use-context kind-dev
Switched to context "kind-dev".

ysmoon@YSMOON:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE   VERSION
dev-control-plane   Ready    control-plane   2m    v1.35.0
```

## 8. 편의 도구 설치 (선택)

여러 클러스터를 자주 오갈 때 다음 도구들이 매우 편리합니다.

### kubectx / kubens

```bash
sudo apt install -y kubectx
```

```bash
ysmoon@YSMOON:~$ kubectx
kind-dev
kind-ha
kind-multi
kind-staging

ysmoon@YSMOON:~$ kubectx kind-dev
Switched to context "kind-dev".
```

### k9s (터미널 대시보드)

```bash
curl -sS https://webinstall.dev/k9s | bash
source ~/.config/envman/PATH.env
```

```bash
ysmoon@YSMOON:~$ k9s
# TUI 대시보드가 실행됩니다 (q로 종료)
```

## 9. 자원 관리

여러 클러스터를 동시에 띄우면 메모리가 빠듯할 수 있습니다. 사용하지 않는 쪽은 멈춰두는 것이 권장됩니다.

### 클러스터 일시 중지/재개

kind는 별도의 cluster stop/start 명령이 없으므로 Docker 컨테이너 단위로 제어합니다.

```bash
# 특정 클러스터의 모든 노드 중지
docker stop $(docker ps -q --filter "label=io.x-k8s.kind.cluster=multi")

# 재개
docker start $(docker ps -aq --filter "label=io.x-k8s.kind.cluster=multi")
```

### 클러스터 완전 삭제

```bash
kind delete cluster --name multi
# 또는 모든 클러스터 한 번에 (서브커맨드가 'clusters' 복수형인 점 주의)
kind delete clusters --all
```

```bash
ysmoon@YSMOON:~$ kind delete clusters --all
Deleting cluster "dev" ...
Deleting cluster "ha" ...
Deleting cluster "multi" ...
Deleting cluster "staging" ...
```

## 10. 다음 단계

멀티노드 클러스터 환경이 준비되어 단일 노드부터 HA 시뮬레이션까지 자유롭게 실험할 수 있습니다. 마지막으로 다음 문서에서 운영급 HA의 이론 배경을 학습하세요.

* **운영급 HA 이론 (마스터 3 + 워커 2)** → [A.4 쿠버네티스 최소 이중화(HA) 구성](A.4%20쿠버네티스%20최소%20이중화(HA)%20구성.md)
  * 위 [5절](#5-ha-시뮬레이션-control-plane-3--worker-2)에서 만든 `kind-ha` 클러스터로 etcd Raft 합의 동작과 워커 재스케줄링을 직접 검증할 수 있습니다.

이 부록(A장)의 학습이 끝나면, 책의 본문 흐름인 **3장 파드**부터 본격적인 K8s 리소스 학습이 시작됩니다.

## 11. 환경 정리 (선택)

상황에 따라 단계별로 정리할 수 있습니다.

### 방법 A: 클러스터만 삭제 (kind/Docker는 유지)

```bash
kind delete clusters --all
```

### 방법 B: kind와 kubectl 제거 (Docker는 유지)

```bash
kind delete clusters --all
sudo rm /usr/local/bin/kind
sudo rm /usr/local/bin/kubectl
```

### 방법 C: Docker Engine까지 제거

```bash
sudo apt remove -y docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
sudo rm -rf /etc/docker
```

### 방법 D: Ubuntu 통째로 삭제 (가장 빠르고 확실)

WSL2 학습 환경이라면 Ubuntu 배포판 자체를 통째로 지우는 것이 가장 빠릅니다. kind·Docker·kubeconfig가 한꺼번에 사라집니다.

```bash
exit
```

```powershell
wsl --unregister Ubuntu
```

> ⚠️ `wsl --unregister`는 **되돌릴 수 없습니다.** Ubuntu 안에 둔 코드·매니페스트·PVC 데이터가 모두 사라지므로, 보존할 파일이 있다면 미리 Windows 쪽(`/mnt/c/...`)이나 Git에 백업해 두세요.
