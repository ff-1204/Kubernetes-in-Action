# Kubernetes in Action 학습 노트

**Kubernetes in Action, Second Edition** (Marko Lukša, Kevin Conner / Manning)을 공부하며 정리한 노트입니다.

- 실습은 [00장. 실습 환경 구축](<00장. 실습 환경 구축/0.1 실습 환경 구축.md>)을 마친 상태를 전제로 합니다.
- 원서 3장(첫 애플리케이션 배포)은 다루지 않아 **장 번호가 원서와 다릅니다.**

| 이 저장소 | 원서 |
| :--- | :--- |
| 03장 | 4장 |
| 04장 | 5장 (5.1~5.3) |
| 05장 | 5장 (5.4~5.5) + 4.3 |
| 06장 이후 | 같은 번호 |

## 작성 원칙

- **모든 자료는 작성한 뒤 공식 문서로 검증합니다.** 쿠버네티스는 <https://kubernetes.io/docs/>, 그 밖의 도구는 각 프로젝트의 공식 문서가 기준입니다.
- **검증에 쓴 출처는 문서 하단에 남깁니다.** 확인하지 못한 내용은 사실처럼 쓰지 않습니다.

## 목차

### 기초 — 환경 구축부터 파드·레이블까지 (00~07장)

| 장 | 제목 | 원제 | 상태 |
| :--- | :--- | :--- | :---: |
| [00](<00장. 실습 환경 구축>) | 실습 환경 구축 | — | ✅ |
| [01](<01장. 쿠버네티스 소개>) | 쿠버네티스 소개 | Introducing Kubernetes | ✅ |
| [02](<02장. 컨테이너 소개>) | 컨테이너 소개 | Understanding containers and containerized applications | ✅ |
| [03](<03장. 쿠버네티스 API와 오브젝트 모델 살펴보기>) | 쿠버네티스 API와 오브젝트 모델 살펴보기 | Navigating the Kubernetes API and object model | ✅ |
| [04](<04장. 파드로 애플리케이션 실행하기>) | 파드로 애플리케이션 실행하기 | Running applications with Pods | ✅ |
| [05](<05장. 다중 컨테이너 파드와 객체 상태>) | 다중 컨테이너 파드와 객체 상태 | Running applications with Pods (계속) | ✅ |
| [06](<06장. 파드 생명주기와 컨테이너 상태 관리>) | 파드 생명주기와 컨테이너 상태 관리 | Managing the Pod life cycle and container health | ✅ |
| [07](<07장. 네임스페이스와 레이블로 리소스 구성하기>) | 네임스페이스와 레이블로 리소스 구성하기 | Organizing Pods and other resources using namespaces and labels | ✅ |

### 1묶음 — 설정과 저장소 (08~10장)

| 장 | 제목 | 원제 | 상태 |
| :--- | :--- | :--- | :---: |
| [08](<08장. ConfigMap과 Secret으로 애플리케이션 설정하기>) | ConfigMap과 Secret으로 애플리케이션 설정하기 | Configuring applications with ConfigMaps and Secrets | ✅ |
| [09](<09장. 스토리지와 설정, 메타데이터를 위한 볼륨 추가하기>) | 스토리지와 설정, 메타데이터를 위한 볼륨 추가하기 | Adding volumes for storage, configuration, and metadata | ✅ |
| [10](<10장. PersistentVolume으로 데이터 영속화하기>) | PersistentVolume으로 데이터 영속화하기 | Persisting data with PersistentVolumes | ✅ |

### 2묶음 — 네트워킹 (11~13장)

| 장 | 제목 | 원제 | 상태 |
| :--- | :--- | :--- | :---: |
| [11](<11장. 서비스로 파드 노출하기>) | 서비스로 파드 노출하기 | Exposing Pods with Services | ✅ |
| [12](<12장. 인그레스로 서비스에 트래픽 라우팅하기>) | 인그레스로 서비스에 트래픽 라우팅하기 | Using Ingress to route traffic to Services | ✅ |
| [13](<13장. Gateway API로 트래픽 라우팅하기>) | Gateway API로 트래픽 라우팅하기 | Routing traffic using the Gateway API | ✅ |

### 3묶음 — 워크로드 컨트롤러 (14~16장)

| 장 | 제목 | 원제 | 상태 |
| :--- | :--- | :--- | :---: |
| 14 | 레플리카셋으로 파드 확장하고 유지하기 | Scaling and maintaining Pods with ReplicaSets | ⬜ |
| 15 | 디플로이먼트로 애플리케이션 업데이트 자동화하기 | Automating application updates with Deployments | ⬜ |
| 16 | 스테이트풀셋으로 상태 유지 애플리케이션 다루기 | Handling stateful applications with StatefulSets | ⬜ |

### 4묶음 — 노드별·배치 워크로드 (17~18장)

| 장 | 제목 | 원제 | 상태 |
| :--- | :--- | :--- | :---: |
| 17 | 데몬셋으로 노드별 워크로드 배포하기 | Deploying per-node workloads with DaemonSets | ⬜ |
| 18 | 잡과 크론잡으로 배치 처리하기 | Batch processing with Jobs and CronJobs | ⬜ |

> ✅ 완료 · ⬜ 시작 전

그림 출처: 쿠버네티스 공식 문서 (CC BY 4.0) — [images/README.md](images/README.md)
