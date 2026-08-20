# Kubernetes in Action

**Kubernetes in Action, Second Edition** (Marko Lukša, Kevin Conner / Manning) 학습 노트입니다.

- 원서: 18장, 5부 구성
- 저자 공식 예제 저장소: <https://github.com/luksa/kubernetes-in-action-2nd-edition>
- 실습 환경 구축: [SETUP.md](SETUP.md) — 2장부터의 실습은 이 문서대로 환경을 갖춘 상태를 전제로 합니다
- 작업 규칙·환경 정보: [CLAUDE.md](CLAUDE.md)

## 목차

| 장 | 제목 | 원제 | 상태 |
| :--- | :--- | :--- | :---: |
| [01](<01장. 쿠버네티스 소개>) | 쿠버네티스 소개 | Introducing Kubernetes | ✅ |
| [02](<02장. 컨테이너 소개>) | 컨테이너 소개 | Understanding containers and containerized applications | ✅ |
| [03](<03장. 쿠버네티스 API와 오브젝트 모델 살펴보기>) | 쿠버네티스 API와 오브젝트 모델 살펴보기 | Navigating the Kubernetes API and object model | ✅ |
| [04](<04장. 파드로 애플리케이션 실행하기>) | 파드로 애플리케이션 실행하기 | Running applications with Pods | ✅ |
| [05](<05장. 다중 컨테이너 파드와 객체 상태>) | 다중 컨테이너 파드와 객체 상태 | Running applications with Pods (계속) | ✅ |
| [06](<06장. 파드 생명주기와 컨테이너 상태 관리>) | 파드 생명주기와 컨테이너 상태 관리 | Managing the Pod life cycle and container health | ✅ |
| [07](<07장. 네임스페이스와 레이블로 리소스 구성하기>) | 네임스페이스와 레이블로 리소스 구성하기 | Organizing Pods and other resources using namespaces and labels | ⬜ |
| [08](<08장. ConfigMap과 Secret으로 애플리케이션 설정하기>) | ConfigMap과 Secret으로 애플리케이션 설정하기 | Configuring applications with ConfigMaps and Secrets | ⬜ |
| [09](<09장. 스토리지와 설정, 메타데이터를 위한 볼륨 추가하기>) | 스토리지와 설정, 메타데이터를 위한 볼륨 추가하기 | Adding volumes for storage, configuration, and metadata | ⬜ |
| [10](<10장. PersistentVolume으로 데이터 영속화하기>) | PersistentVolume으로 데이터 영속화하기 | Persisting data with PersistentVolumes | ⬜ |
| [11](<11장. 서비스로 파드 노출하기>) | 서비스로 파드 노출하기 | Exposing Pods with Services | ⬜ |
| [12](<12장. 인그레스로 서비스에 트래픽 라우팅하기>) | 인그레스로 서비스에 트래픽 라우팅하기 | Using Ingress to route traffic to Services | ⬜ |
| [13](<13장. Gateway API로 트래픽 라우팅하기>) | Gateway API로 트래픽 라우팅하기 | Routing traffic using the Gateway API | ⬜ |
| [14](<14장. 레플리카셋으로 파드 확장하고 유지하기>) | 레플리카셋으로 파드 확장하고 유지하기 | Scaling and maintaining Pods with ReplicaSets | ⬜ |
| [15](<15장. 디플로이먼트로 애플리케이션 업데이트 자동화하기>) | 디플로이먼트로 애플리케이션 업데이트 자동화하기 | Automating application updates with Deployments | ⬜ |
| [16](<16장. 스테이트풀셋으로 상태 유지 애플리케이션 다루기>) | 스테이트풀셋으로 상태 유지 애플리케이션 다루기 | Handling stateful applications with StatefulSets | ⬜ |
| [17](<17장. 데몬셋으로 노드별 워크로드 배포하기>) | 데몬셋으로 노드별 워크로드 배포하기 | Deploying per-node workloads with DaemonSets | ⬜ |
| [18](<18장. 잡과 크론잡으로 배치 처리하기>) | 잡과 크론잡으로 배치 처리하기 | Batch processing with Jobs and CronJobs | ⬜ |

> 상태: ⬜ 시작 전 · 🟨 진행 중 · ✅ 완료 · ↪ 다른 문서로 통합

**장 번호가 원서와 다릅니다.** 원서 3장(첫 애플리케이션 배포)은 맛보기 성격이라 통째로 걷어내고, 그만큼 뒤 장을 당겼습니다.

| 이 저장소 | 원서 | 비고 |
| :--- | :--- | :--- |
| — | 3장 | 클러스터 구축은 [SETUP.md](SETUP.md)로. 배포·노출·확장은 04장·11장·14~15장에서 제대로 다룸 |
| 03장 | 4장 | |
| 04장 | 5장 (5.1~5.3) | |
| 05장 | 5장 (5.4~5.5) + 4장 (4.3 객체 상태와 이벤트) | 상태·이벤트는 파드를 다뤄 본 뒤라야 와닿아서 뒤로 옮김 |
| 06장~18장 | 6장~18장 | 번호 동일 |

## 노트 작성 규칙

- 파일 하나 = 책의 한 개 절 (예: `1.1 쿠버네티스가 필요한 이유.md`)
- 제목은 `# <절번호> <절 제목>`
- 개념은 `###` 소제목 + 불릿으로 전개하고, 핵심 용어는 **볼드**에 영문 병기
- 문서 끝에 `## 요약 및 비유` 표(개념 / 비유 / 기술적 의미)와 `## 결론` 정리
