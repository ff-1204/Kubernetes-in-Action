# A.2 쿠버네티스 클러스터 설치(kind)

## 1. WSL2 환경 준비

먼저 PowerShell(관리자 권한)에서 WSL2와 Ubuntu가 설치되어 있는지 확인합니다.

```powershell
wsl --list --verbose
```

배포판이 하나도 없으면 아래와 같이 안내 메시지가 출력됩니다.

```powershell
PS C:\windows\system32> wsl --list --verbose
Linux용 Windows 하위 시스템 설치된 배포가 없습니다.
아래 지침에 따라 배포를 설치하여 이 resolve 수 있습니다.

사용 가능한 배포를 나열하려면 'wsl.exe --list --online' 사용
설치하려면 'wsl.exe --install <Distro>'를 선택하세요.
```

Ubuntu가 없다면 설치합니다. `--no-launch`는 자동 실행을 건너뛰며, 사용자 계정 생성은 2절에서 진행합니다.

```powershell
wsl --install -d Ubuntu --no-launch
```

설치가 끝났으면 WSL 버전이 2인지 확인합니다.

```powershell
PS C:\windows\system32> wsl --list --verbose
  NAME      STATE           VERSION
* Ubuntu    Stopped         2
```

VERSION이 1로 나온다면 다음 명령으로 변환합니다.

```powershell
wsl --set-version Ubuntu 2
```

### WSL2 메모리 한도 조정 (권장)

기본 WSL2 메모리 상한은 **호스트 RAM의 50% 또는 8GB 중 작은 값**입니다. kind는 노드 하나당 컨테이너를 띄우므로 멀티노드 구성에서는 기본 한도가 부족할 수 있습니다. 노트북 메모리가 16GB 이상이라면 한도를 늘려두는 것이 좋습니다.

PowerShell에서 사용자 폴더의 `.wslconfig`를 엽니다.

```powershell
notepad $env:USERPROFILE\.wslconfig
```

다음 내용으로 저장합니다(예: 12GB 할당).

```ini
[wsl2]
memory=12GB
processors=4
swap=4GB
```

설정을 적용하려면 WSL을 한 번 재시작합니다.

```powershell
wsl --shutdown
```

## 2. WSL2 내 사전 설정

이후 명령은 모두 WSL2(Ubuntu) 안에서 실행해야 합니다. PowerShell에서 `wsl`을 입력해 Ubuntu 셸로 진입합니다.

아래 로그는 **최초 실행 시 사용자 계정 설정**입니다.

```bash
PS C:\windows\system32> wsl
Provisioning the new WSL instance Ubuntu
This might take a while...
Create a default Unix user account: ysmoon
New password:
Retype new password:
passwd: password updated successfully
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ysmoon@YSMOON:/mnt/c/windows/system32$
```

패키지를 업데이트하고 필수 도구를 설치합니다.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl
```

### systemd 활성화 검증

Docker Engine은 **systemd 서비스로 등록·실행**됩니다. WSL2의 systemd 지원은 Windows 11(빌드 22000 이상)부터 가능하지만, 구버전 사용자나 일부 배포판은 명시적 활성화가 필요합니다.

```bash
ps -p 1 -o comm=
```

`systemd`가 출력되면 활성화된 상태입니다. `init` 등 다른 값이 나오면 다음과 같이 활성화합니다.

```bash
sudo tee /etc/wsl.conf <<EOF
[boot]
systemd=true
EOF
```

PowerShell로 빠져나가 WSL을 재시작합니다.

```bash
exit
```

```powershell
wsl --shutdown
wsl
```

재진입 후 다시 `ps -p 1 -o comm=`로 `systemd`가 떴는지 확인합니다.

## 3. Docker Engine 설치

> ⚠️ **Docker Desktop이 아닙니다.** Docker Desktop은 직원 250명 이상 또는 연 매출 $10M 이상 회사에서 유료입니다. WSL 안에 직접 설치하는 **Docker Engine(docker-ce)**은 Apache 2.0 라이선스로 무료입니다.

공식 설치 스크립트를 실행합니다.

```bash
curl -fsSL https://get.docker.com | sudo sh
```

매번 `sudo docker`를 입력하지 않도록 현재 사용자를 `docker` 그룹에 추가합니다.

```bash
sudo usermod -aG docker $USER
```

그룹 변경을 적용하려면 WSL 셸을 한 번 빠져나갔다가 다시 들어옵니다.

```bash
exit
```

```powershell
wsl
```

동작 확인:

```bash
ysmoon@YSMOON:~$ docker version
Client: Docker Engine - Community
 Version:           27.x.x
Server: Docker Engine - Community
 Engine:
  Version:          27.x.x
```

`permission denied` 에러 없이 Server 정보까지 출력되면 그룹 적용이 성공한 것입니다.

## 4. kind 설치

K8s SIG 공식 도구인 kind를 설치합니다(amd64 기준).

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
sudo install -o root -g root -m 0755 kind /usr/local/bin/kind
rm kind
```

> 특정 버전을 지정하려면 `latest` 자리에 `v0.x.x`를 넣으세요. 릴리스: https://github.com/kubernetes-sigs/kind/releases

설치 확인:

```bash
ysmoon@YSMOON:~$ kind version
kind v0.x.x go1.x.x linux/amd64
```

## 5. kubectl 설치

kind는 kubectl을 포함하지 않으므로 별도로 설치합니다(K8s 공식).

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl
```

확인:

```bash
ysmoon@YSMOON:~$ kubectl version --client
Client Version: v1.x.x
Kustomize Version: v5.x.x
```

## 6. 단일 노드 클러스터 생성

```bash
kind create cluster --name study
```

**생성 로그**

```bash
ysmoon@YSMOON:~$ kind create cluster --name study
Creating cluster "study" ...
 ✓ Ensuring node image (kindest/node:v1.x.x) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-study"
You can now use your cluster with:

kubectl cluster-info --context kind-study

Have a nice day! 👋
```

kind는 클러스터 생성 시 `~/.kube/config`에 컨텍스트(`kind-study`)를 자동으로 추가하고 현재 컨텍스트로 전환합니다.

## 7. 클러스터 동작 확인

```bash
ysmoon@YSMOON:~$ kubectl get nodes
NAME                  STATUS   ROLES           AGE   VERSION
study-control-plane   Ready    control-plane   30s   v1.x.x
```

control-plane 노드가 `Ready` 상태로 나오면 정상입니다.

```bash
ysmoon@YSMOON:~$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:xxxxx
CoreDNS is running at https://127.0.0.1:xxxxx/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```bash
ysmoon@YSMOON:~$ kubectl get pods -A
NAMESPACE            NAME                                          READY   STATUS    RESTARTS   AGE
kube-system          coredns-xxxxxxxxxx-xxxxx                      1/1     Running   0          1m
kube-system          coredns-xxxxxxxxxx-xxxxx                      1/1     Running   0          1m
kube-system          etcd-study-control-plane                      1/1     Running   0          1m
kube-system          kindnet-xxxxx                                 1/1     Running   0          1m
kube-system          kube-apiserver-study-control-plane            1/1     Running   0          1m
kube-system          kube-controller-manager-study-control-plane   1/1     Running   0          1m
kube-system          kube-proxy-xxxxx                              1/1     Running   0          1m
kube-system          kube-scheduler-study-control-plane            1/1     Running   0          1m
local-path-storage   local-path-provisioner-xxxxxxxxxx-xxxxx       1/1     Running   0          1m
```

표준 K8s 컨트롤 플레인 컴포넌트(`etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`)가 모두 `Running` 상태로 보이면 성공입니다.

## 8. 책 예제용 Ingress 컨트롤러 설치 (선택)

책의 Ingress 예제(보통 `nginx-ingress`)를 그대로 따라가려면 별도 설치가 필요합니다. kind는 Ingress 컨트롤러를 기본 포함하지 않습니다.

> ⚠️ **주의**: 이 절을 실행하면 **6절에서 만든 클러스터(`kind-study`)가 삭제되고 새로 만들어집니다.** 그 안에 띄워둔 Pod·Deployment·Service 등은 모두 사라집니다. 6절 결과물을 보존하려면 먼저 매니페스트를 yaml 파일로 export하세요(예: `kubectl get all -o yaml > backup.yaml`). 새 클러스터에서 다시 적용하면 됩니다.

먼저 클러스터 생성 시 80/443 포트 매핑이 필요하므로 기존 클러스터를 삭제하고 다시 만듭니다.

```bash
kind delete cluster --name study

cat <<EOF | kind create cluster --name study --config=-
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

nginx-ingress 설치(kind 전용 매니페스트):

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

이제 책의 Ingress 예제를 그대로 적용해도 동작합니다.

> Ingress가 필요 없는 챕터까지는 6절의 단순한 클러스터로도 충분합니다. 책에서 Ingress가 처음 등장할 때 이 절로 돌아오세요.

## 9. 다음 단계

kind 단일 노드 환경이 준비되었습니다. 이어서 다음 문서로 진행하세요.

* **멀티노드 클러스터 / 다중 클러스터 / HA 시뮬레이션** → [A.3 멀티노드 클러스터 구성(kind)](A.3%20멀티노드%20클러스터%20구성(kind).md)
* **운영급 HA 이론** → [A.4 쿠버네티스 최소 이중화(HA) 구성](A.4%20쿠버네티스%20최소%20이중화(HA)%20구성.md) 에서 "왜 마스터 3 + 워커 2가 운영급 최소인가"의 분산 시스템 원리를 학습합니다.

## 10. 환경 정리 (선택)

상황에 따라 단계별로 정리할 수 있습니다.

### 방법 A: 클러스터만 삭제 (kind/Docker는 유지)

```bash
kind delete cluster --name study
# 또는 모든 클러스터를 한 번에
kind delete cluster --all
```

### 방법 B: kind와 kubectl 제거 (Docker는 유지)

```bash
kind delete cluster --all
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

WSL2 학습 환경이라면 Ubuntu 배포판 자체를 통째로 지우는 것이 가장 빠릅니다.

```bash
exit
```

```powershell
wsl --list --verbose
wsl --unregister Ubuntu
```

> ⚠️ `wsl --unregister`는 **되돌릴 수 없습니다.** Ubuntu 안에 둔 코드·매니페스트·PVC 데이터가 모두 사라지므로, 보존할 파일이 있다면 미리 Windows 쪽(`/mnt/c/...`)이나 Git에 백업해 두세요.
