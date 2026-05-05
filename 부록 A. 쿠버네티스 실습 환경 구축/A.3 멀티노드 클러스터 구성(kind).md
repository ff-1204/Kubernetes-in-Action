# A.3 멀티노드 클러스터 구성(kind)

[A.2 쿠버네티스 클러스터 설치(kind)](A.2%20쿠버네티스%20클러스터%20설치(kind).md) 까지 완료된 상태에서 시작합니다. kind는 단일 도구로 단일 노드부터 멀티노드, 다중 클러스터, HA 시뮬레이션까지 모두 지원하므로 별도 도구 추가가 필요 없습니다.

## 0. kind의 멀티노드 구조

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

## 1. 사전 점검

```bash
ysmoon@YSMOON:~$ docker version
Server: Docker Engine - Community
 Engine:
  Version:          27.x.x
```

```bash
ysmoon@YSMOON:~$ kind version
kind v0.x.x go1.x.x linux/amd64
```

```bash
ysmoon@YSMOON:~$ kubectl version --client
Client Version: v1.x.x
```

A.2에서 만든 단일 노드 클러스터가 남아 있다면 정리합니다.

```bash
kind get clusters
# study

kind delete cluster --name study
```

## 2. 멀티노드 단일 클러스터 (control-plane 1 + worker 2)

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
kind create cluster --name multi --config kind-multi.yaml
```

**생성 로그**

```bash
ysmoon@YSMOON:~$ kind create cluster --name multi --config kind-multi.yaml
Creating cluster "multi" ...
 ✓ Ensuring node image (kindest/node:v1.x.x) 🖼
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
multi-control-plane   Ready    control-plane   2m    v1.x.x
multi-worker          Ready    <none>          90s   v1.x.x
multi-worker2         Ready    <none>          90s   v1.x.x
```

## 3. HA 시뮬레이션 (control-plane 3 + worker 2)

운영급 HA의 etcd Raft 합의를 직접 검증할 수 있는 구성입니다. 이론적 배경은 [A.4 HA 구성](A.4%20쿠버네티스%20최소%20이중화(HA)%20구성.md) 을 참고하세요.

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

kind create cluster --name ha --config kind-ha.yaml
```

확인:

```bash
ysmoon@YSMOON:~$ kubectl get nodes --context kind-ha
NAME                STATUS   ROLES           AGE   VERSION
ha-control-plane    Ready    control-plane   3m    v1.x.x
ha-control-plane2   Ready    control-plane   2m    v1.x.x
ha-control-plane3   Ready    control-plane   2m    v1.x.x
ha-worker           Ready    <none>          1m    v1.x.x
ha-worker2          Ready    <none>          1m    v1.x.x
```

### 합의 동작 검증

마스터 1대를 죽여보면서 quorum 유지를 확인합니다.

```bash
ysmoon@YSMOON:~$ docker stop ha-control-plane2
ha-control-plane2

ysmoon@YSMOON:~$ kubectl get nodes --context kind-ha
NAME                STATUS     ROLES           AGE   VERSION
ha-control-plane    Ready      control-plane   5m    v1.x.x
ha-control-plane2   NotReady   control-plane   4m    v1.x.x
ha-control-plane3   Ready      control-plane   4m    v1.x.x
ha-worker           Ready      <none>          3m    v1.x.x
ha-worker2          Ready      <none>          3m    v1.x.x
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
kubectl get nodes --context kind-ha    # 응답 안 됨 (quorum 깨짐)
docker start ha-control-plane2 ha-control-plane3
```

## 4. 다중 독립 클러스터 (dev / staging 분리)

서로 다른 환경을 동시에 운용하고 싶을 때 유용합니다.

```bash
kind create cluster --name dev
kind create cluster --name staging
```

확인:

```bash
ysmoon@YSMOON:~$ kind get clusters
dev
ha
multi
staging
study
```

각 클러스터는 별도의 Docker 네트워크와 노드 컨테이너를 가지므로 완전히 격리됩니다.

## 5. 컨텍스트 전환

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
dev-control-plane   Ready    control-plane   2m    v1.x.x
```

## 6. 편의 도구 설치 (선택)

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

## 7. 자원 관리

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
# 또는 모든 클러스터 한 번에
kind delete cluster --all
```

```bash
ysmoon@YSMOON:~$ kind delete cluster --all
Deleting cluster "dev" ...
Deleting cluster "ha" ...
Deleting cluster "multi" ...
Deleting cluster "staging" ...
```

## 8. 다음 단계

멀티노드 클러스터 환경이 준비되어 단일 노드부터 HA 시뮬레이션까지 자유롭게 실험할 수 있습니다. 마지막으로 다음 문서에서 운영급 HA의 이론 배경을 학습하세요.

* **운영급 HA 이론 (마스터 3 + 워커 2)** → [A.4 쿠버네티스 최소 이중화(HA) 구성](A.4%20쿠버네티스%20최소%20이중화(HA)%20구성.md)
  * 위 [3절](#3-ha-시뮬레이션-control-plane-3--worker-2)에서 만든 `kind-ha` 클러스터로 etcd Raft 합의 동작과 워커 재스케줄링을 직접 검증할 수 있습니다.

이 부록(A장)의 학습이 끝나면, 책의 본문 흐름인 **3장 파드**부터 본격적인 K8s 리소스 학습이 시작됩니다.

## 9. 환경 정리 (선택)

### 클러스터만 삭제 (kind/Docker는 유지)

```bash
kind delete cluster --all
```

### kind와 kubectl까지 제거 (Docker는 유지)

```bash
kind delete cluster --all
sudo rm /usr/local/bin/kind
sudo rm /usr/local/bin/kubectl
```

### Docker Engine까지 제거

```bash
sudo apt remove -y docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
sudo rm -rf /etc/docker
```

### Ubuntu 통째로 삭제

가장 빠르고 확실한 방법입니다. kind·Docker·kubeconfig가 한꺼번에 사라집니다.

```bash
exit
```

```powershell
wsl --unregister Ubuntu
```

> ⚠️ `wsl --unregister`는 **되돌릴 수 없습니다.** Ubuntu 안에 둔 코드·매니페스트·PVC 데이터가 모두 사라지므로, 보존할 파일이 있다면 미리 Windows 쪽(`/mnt/c/...`)이나 Git에 백업해 두세요.
