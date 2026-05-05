# A.2 쿠버네티스 클러스터 설치(kind)

> **WSL2 + Docker Engine + kind + kubectl** 조합으로 단일 노드 K8s 클러스터(표준 K8s 학습 환경)를 구축합니다.

## 1. WSL2 환경 준비

먼저 PowerShell(관리자 권한)에서 WSL2와 Ubuntu가 설치되어 있는지 확인합니다.

```powershell
wsl --list --verbose
```

배포판이 하나도 없으면 아래와 같이 안내 메시지가 출력됩니다.

```powershell
PS C:\windows\system32> wsl --list --verbose
Linux용 Windows 하위 시스템에 설치된 배포가 없습니다.
아래 지침에 따라 배포를 설치하여 해결할 수 있습니다.

사용 가능한 배포를 나열하려면 'wsl.exe --list --online' 사용
설치하려면 'wsl.exe --install <Distro>'를 선택하세요.
```

Ubuntu가 없다면 설치합니다.

```powershell
wsl --install -d Ubuntu --no-launch
```

설치가 완료되면 아래와 같이 안내 메시지가 출력됩니다.

```powershell
PS C:\windows\system32> wsl --install -d Ubuntu --no-launch
다운로드 중: Ubuntu
설치 중: Ubuntu
배포가 설치되었습니다. 'wsl.exe -d Ubuntu'을(를) 통해 시작할 수 있습니다.
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

## 2. WSL2 내 사전 설정 (사용자 권한 필요)

아래 로그는 **최초 실행 시 사용자 계정 설정 + 진입까지**입니다.

```text
PS C:\windows\system32> wsl
Provisioning the new WSL instance Ubuntu
This might take a while...
Create a default Unix user account: ysmoon
New password:
Retype new password:
passwd: password updated successfully
usermod: no changes
Welcome to Ubuntu 26.04 LTS (GNU/Linux 6.6.87.2-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://docs.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue May  5 21:31:41 KST 2026

  System load:  0.08                Processes:             42
  Usage of /:   0.1% of 1006.85GB   Users logged in:       0
  Memory usage: 3%                  IPv4 address for eth0: 172.26.201.12
  Swap usage:   0%


This message is shown once a day. To disable it please create the
/home/ysmoon/.hushlogin file.
ysmoon@YSMOON:/mnt/c/windows/system32$
```

### 패키지 업데이트

패키지를 업데이트하고 `curl`을 설치합니다(이후 단계에서 kind·kubectl 바이너리를 내려받는 데 사용).

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl
```

## 3. Docker Engine 설치

> ⚠️ **Docker Desktop이 아닙니다.** Docker Desktop은 직원 250명 이상 또는 연 매출 $10M 이상 회사에서 유료입니다. WSL 안에 직접 설치하는 **Docker Engine(docker-ce)**은 Apache 2.0 라이선스로 무료입니다.

공식 설치 스크립트를 실행합니다. 스크립트 실행 중 Ctrl+C를 누르면 취소됩니다.

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
 Version:           29.x.x
Server: Docker Engine - Community
 Engine:
  Version:          29.x.x
```

`permission denied` 에러 없이 Server 정보까지 출력되면 그룹 적용이 성공한 것입니다.

## 4. kind 설치 (사용자 권한 필요)

K8s SIG 공식 도구인 kind를 설치합니다(amd64 기준).

> **Kind의 안정 버전(`v0.31.0`)을 명시적으로 지정**합니다. 이 버전은 Kubernetes `v1.35.0`을 기본 노드 이미지로 채택합니다.

```bash
ysmoon@YSMOON:/mnt/c/windows/system32$ cd ~
ysmoon@YSMOON:~$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-amd64
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     97 100     97   0      0     99      0                              0
  0      0   0      0   0      0      0      0           00:01              0
100 10.58M 100 10.58M   0      0  4.38M      0   00:02   00:02         10.26M
ysmoon@YSMOON:~$ sudo install -o root -g root -m 0755 kind /usr/local/bin/kind
ysmoon@YSMOON:~$ rm kind
```

설치 확인:

```bash
ysmoon@YSMOON:~$ kind version
kind v0.31.0 go1.25.5 linux/amd64
ysmoon@YSMOON:~$
```

## 5. kubectl 설치

kind는 kubectl을 포함하지 않으므로 별도로 설치합니다(K8s 공식).

```bash
ysmoon@YSMOON:~$ sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100 56.74M 100 56.74M   0      0 10.98M      0   00:05   00:05         10.97M
ysmoon@YSMOON:~$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
ysmoon@YSMOON:~$ rm kubectl
rm: remove write-protected regular file 'kubectl'? y
ysmoon@YSMOON:~$
```

확인:

```bash
ysmoon@YSMOON:~$ kubectl version --client
Client Version: v1.36.0
Kustomize Version: v5.8.1
```

> 💡 kubectl은 `stable.txt`가 가리키는 **최신 안정 버전**이 깔리므로 클러스터(`v1.35.0`)보다 1 마이너 위(`v1.36.0`)가 받아질 수 있습니다. kubectl은 ±1 마이너 버전 호환을 보장하므로 문제 없습니다.

## 6. 단일 노드 클러스터 생성(Kubernetes v1.35.0 적용)

```bash
kind create cluster --image kindest/node:v1.35.0 --name ysmoon
```

**생성 로그**

```bash
ysmoon@YSMOON:~$ kind create cluster --image kindest/node:v1.35.0 --name ysmoon
Creating cluster "ysmoon" ...
 ✓ Ensuring node image (kindest/node:v1.35.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-ysmoon"
You can now use your cluster with:

kubectl cluster-info --context kind-ysmoon

Have a nice day! 👋
```

## 7. 클러스터 동작 확인

세 가지 명령으로 클러스터가 정상 동작하는지 단계별로 확인합니다.

> ⏳ **클러스터 생성 직후 30~60초**는 노드가 `NotReady`, 일부 파드가 `Pending`으로 보이는 게 정상입니다. CNI(`kindnet`) 초기화와 시스템 파드 기동에 시간이 필요하기 때문입니다. 잠시 기다린 뒤 아래 명령들을 다시 조회해 보세요.
>
> ```text
> # kubectl get nodes (직후)
> ysmoon-control-plane   NotReady   control-plane   14s   v1.35.0
>
> # kubectl get pods -A (직후)
> coredns-7d764666f9-...   0/1   Pending   0   11s
> ```

### API 서버 접속 확인

API 서버(컨트롤 플레인)와 클러스터 내부 DNS의 엔드포인트가 출력되면 클러스터가 응답하고 있다는 뜻입니다.

```bash
ysmoon@YSMOON:~$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:xxxxx
CoreDNS is running at https://127.0.0.1:xxxxx/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### 노드 상태 확인

클러스터에 등록된 노드(서버) 목록과 상태를 출력합니다. control-plane 노드가 `Ready` 상태로 나오면 정상입니다.

```bash
ysmoon@YSMOON:~$ kubectl get nodes
NAME                   STATUS   ROLES           AGE     VERSION
ysmoon-control-plane   Ready    control-plane   5m59s   v1.35.0
```

### 시스템 파드 확인

`kube-system` 네임스페이스에서 컨트롤 플레인 컴포넌트 파드들이 모두 `Running` 상태인지 확인합니다.

```bash
ysmoon@YSMOON:~$ kubectl get pods -A
NAMESPACE            NAME                                           READY   STATUS    RESTARTS   AGE
kube-system          coredns-7d764666f9-stwtk                       1/1     Running   0          8m5s
kube-system          coredns-7d764666f9-ts57x                       1/1     Running   0          8m5s
kube-system          etcd-ysmoon-control-plane                      1/1     Running   0          8m10s
kube-system          kindnet-wspvh                                  1/1     Running   0          8m5s
kube-system          kube-apiserver-ysmoon-control-plane            1/1     Running   0          8m10s
kube-system          kube-controller-manager-ysmoon-control-plane   1/1     Running   0          8m10s
kube-system          kube-proxy-xhwwb                               1/1     Running   0          8m5s
kube-system          kube-scheduler-ysmoon-control-plane            1/1     Running   0          8m12s
local-path-storage   local-path-provisioner-67b8995b4b-bz8q4        1/1     Running   0          8m5s
```

표준 K8s 컨트롤 플레인 컴포넌트(`etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`)가 모두 `Running` 상태로 보이면 성공입니다.

---

## 8. 환경 정리 (선택)

상황에 따라 단계별로 정리할 수 있습니다.

### 방법 A: 클러스터만 삭제 (kind/Docker는 유지)

```bash
kind delete cluster --name ysmoon
# 또는 모든 클러스터를 한 번에 (서브커맨드가 'clusters' 복수형인 점 주의)
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

WSL2 학습 환경이라면 Ubuntu 배포판 자체를 통째로 지우는 것이 가장 빠릅니다.

```bash
exit
```

```powershell
wsl --list --verbose
wsl --unregister Ubuntu
```

> ⚠️ `wsl --unregister`는 **되돌릴 수 없습니다.** Ubuntu 안에 둔 코드·매니페스트·PVC 데이터가 모두 사라지므로, 보존할 파일이 있다면 미리 Windows 쪽(`/mnt/c/...`)이나 Git에 백업해 두세요.
