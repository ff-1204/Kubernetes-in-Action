# CLAUDE.md

**Kubernetes in Action, Second Edition** (Marko Lukša, Kevin Conner / Manning) 학습 노트 저장소입니다.
이 문서는 다음 세션에서 작업을 이어갈 때 읽는 안내서입니다. 목차와 진행 상태는 [README.md](README.md)에 있습니다.

---

## 1. 저장소 구조

```
Kubernetes-in-Action/
├── README.md                       # 목차 + 진행 상태 + 노트 작성 규칙
├── CLAUDE.md                       # 이 문서
├── SETUP.md                        # 실습 환경 구축 절차 + 배경 개념 + kubectl/kubeconfig 설정
├── kind-config.yaml                # 실습 클러스터 설정 (SETUP.md에서 사용)
├── images/                         # 쿠버네티스 공식 문서 이미지 (CC BY 4.0)
│   └── README.md                   # 이미지별 원본 URL·라이선스 고지
├── 01장. 쿠버네티스 소개/
│   ├── 1.0 쿠버네티스를 배우기 전에.md  # 원서에 없는 선수 지식 보충 (§3 참고)
│   ├── 1.1 쿠버네티스 소개.md
│   ├── 1.2 쿠버네티스 이해하기.md   # 1.3(환경구축)은 SETUP.md로 통합
│   └── question/                   # 면접 대비 질문·답변 (§3 참고)
│       ├── 01장 핵심 질문.md        # 질문만 — 먼저 스스로 답해 보는 용도
│       └── 01장 핵심 질문 답변.md
├── 02장. ~ 04장. ...               # 노트 + question/ (장 번호가 원서와 다름 — §3 참고)
├── 05장. ~ 06장. ...               # 노트만 — question/ 아직 없음
└── 07장. ~ 18장. ...               # .gitkeep 만 있는 빈 폴더
```

- 빈 장 폴더에는 `.gitkeep`이 있습니다. **문서를 작성하면 `.gitkeep`을 삭제**합니다(1~6장에는 없음).
- 새 장을 완성하면 **README.md 목차의 상태를 `⬜` → `✅`** 로 바꿉니다.
- **`question/` 폴더는 노트와 별개입니다.** 노트를 고칠 때 질문·답변도 같이 맞춰 주세요. 규칙은 §3 마지막에 있습니다.

## 2. 진행 상태 (2026-08-29 기준)

| 장 | 노트 | `question/` |
| :--- | :--- | :--- |
| 01 ~ 04장 | ✅ 작성 완료 | ✅ 질문 117문항 + 답변 |
| 05 ~ 06장 | ✅ 작성 완료 | ⬜ 없음 |
| 07장 이후 | ⬜ 미작성 | ⬜ 없음 |

**다음 작업: 07장 네임스페이스와 레이블로 리소스 구성하기**

> **7장에서 꼭 짚을 것 — 리눅스 네임스페이스와의 구분.**
> 이름만 같고 무관한 개념이라 가장 흔히 혼동하는 지점입니다. [2.3절](<02장. 컨테이너 소개/2.3 컨테이너를 가능하게 하는 기술.md>)에 이미 비교표가 있으니, 7장 도입부에서 반대 방향으로 다시 확인시킵니다.
> * 쿠버네티스 네임스페이스는 **커널 격리가 아니라 오브젝트를 묶는 이름 구역**(폴더에 가까움)
> * `dev`/`prod`로 나눠도 **커널 수준 격리 강도는 같고**, 기본 설정에서는 **네임스페이스가 달라도 네트워크로 서로 접근됩니다.** 막으려면 NetworkPolicy가 필요
> * 리눅스 네임스페이스를 공유하는 단위는 오직 **한 파드 안의 컨테이너들**

---

## 3. 노트 작성 규칙

README.md의 "노트 작성 규칙"이 원칙이고, 아래는 실제 문서에서 굳어진 세부 관례입니다.

### 파일과 제목

- **파일 하나 = 책의 한 개 절.** 파일명은 `<절번호> <절 제목>.md` (예: `6.2 라이브니스 프로브로 컨테이너 살려 두기.md`)
- 문서 제목은 `# <절번호> <절 제목>`
- **절 구성은 원서 목차를 따릅니다.** 임의로 쪼개거나 합치지 않습니다. 목차가 불확실하면 확인 후 작성합니다.
- **예외 — `1.0 쿠버네티스를 배우기 전에.md`.** 원서에 없는 유일한 보충 절입니다. 스터디에서 쿠버네티스가 아니라 **그 아래 깔린 기초 용어**(서버/노드/클러스터, 프로세스, IP·포트·DNS, HTTP 상태 코드, 리눅스 경로, YAML 문법, 무상태/상태 유지)에서 막혀서 앞에 모았습니다. **여기는 개념 소개만 하고 본론은 각 장에 넘깁니다** — VM 비교표는 2.1, 리눅스 네임스페이스는 2.3, kubectl·선언형은 3.1, `apiVersion/kind/metadata/spec` 틀은 3장이 전담합니다. 이 원칙을 깨고 1.0을 불리지 마세요.
- **예외 — 환경 구축 절은 쓰지 않습니다.** 원서의 1.3(kind 환경구축)과 3.1(클러스터 배포하기)에 해당하는 내용은 전부 [SETUP.md](SETUP.md)로 통합했습니다. 앞으로도 설치·클러스터 생성 이야기가 나오면 노트에 다시 쓰지 말고 SETUP.md에 넣거나 링크합니다.
- **3.1에서 일부러 미룬 것 — 나중에 꼭 채웁니다.** 3.1은 개념만 잡는 절이라 아래를 뺐습니다. 해당 장을 쓸 때 다뤄야 합니다.
  - **네임스페이스 소속 vs 클러스터 전역** 객체 구분(`kubectl api-resources --namespaced`) → **7장**
  - **`kubectl explain`** (내장 매뉴얼, `-required-` 표시, OpenAPI v3) → **4.2 YAML로 파드 만들기**. 현재 저장소 어디에도 없습니다
  - **CRD** 상세(등록·사용 예시) → **16장 오퍼레이터**
- **kubectl 명령의 분담.** `3.1`은 **개념만** 다룹니다 — "모든 kubectl 명령은 API 요청 하나다"(`-v=8`)와 **명령형 vs 선언형**까지. **실제 명령 사용법은 4장이 전담**합니다(`apply`·`get`·`describe`는 4.2, `logs`·`exec`·`cp`·`debug`는 4.3). 3.1에 명령 목록을 다시 늘어놓지 마세요.
- **장 번호가 원서와 다릅니다.** 원서 3장(첫 애플리케이션 배포)을 통째로 걷어내고 뒤 장을 당겼습니다. 대응표는 [README.md](README.md) 목차 아래에 있습니다. 원서 3장의 내용은 클러스터 배포 → [SETUP.md](SETUP.md), kubectl → 3.1절, 배포·노출·확장 → 04장·11장·14~15장으로 흩어졌고, 3.3~3.4에 쓰던 이미지 4개는 `images/`에 남겨 `images/README.md`에 *미사용 — ○장 예정* 으로 표시했습니다. **"첫 애플리케이션 배포" 장을 다시 만들지 마세요.**

### 본문

- 개념은 `###` 소제목 + 불릿으로 전개, 핵심 용어는 **볼드**에 영문 병기 — 예: **선언적 모델(Declarative Model)**
- 비교는 표로 정리
- 명령어는 언어를 지정한 코드 블록(` ```bash `, ` ```yaml `, ` ```powershell `)
- 실행 결과 예시는 **실제로 실행해서 얻은 값**을 씁니다. 지어내지 않습니다
- **컨테이너 명령은 `docker`가 아니라 `podman`으로 씁니다.** 책은 Docker 기준이지만 실습 환경에는 Podman만 설치되어 있습니다. 이미지는 `docker.io/library/busybox`처럼 **레지스트리 주소까지** 적습니다(Podman은 짧은 이름을 되묻습니다). 다만 *역사·개념 설명*에서의 Docker 언급(2.1절 등)은 그대로 둡니다
- 함정·주의사항은 `>` 인용 블록으로 강조
- 다른 장으로 이어지는 개념은 `(11장에서 다룹니다)`처럼 명시

### 문서 끝 고정 구성

```markdown
## 요약 및 비유
| 개념 | ○○ 비유 | 기술적 의미 |     ← 비유 주제는 문서마다 다르게 (급식실, 택배, 병원 진료 등)

## 결론
* 3~5개 불릿으로 핵심 정리

## 공식 문서 업데이트 (2026년 8월 기준)
* 책(1.24~1.27 시절)과 현재가 달라진 부분
* 참고: <https://kubernetes.io/docs/...>
```

`공식 문서 업데이트` 절은 **책과 현재 동작이 실제로 다를 때만** 씁니다. 버전 번호를 쓸 때는 반드시 공식 문서로 확인하고, 확인 못 하면 버전을 빼고 서술합니다.

### 면접 대비 질문 — `question/` 폴더

각 장 폴더 안에 `question/`을 두고 **질문 파일과 답변 파일을 한 쌍**으로 만듭니다. **"파일 하나 = 책의 한 개 절" 규칙의 예외**입니다. 현재 01~04장에 있습니다.

| 파일 | 내용 |
| :--- | :--- |
| `NN장 핵심 질문.md` | 질문만. 먼저 스스로 답해 보는 용도 |
| `NN장 핵심 질문 답변.md` | 답변 |

**질문 파일**

- 맨 앞에 **`## 이 장에서 익힐 개념`** 표를 둡니다. 그 장을 관통하는 축 **7개 안팎**만 한 줄 정의로. 개별 사실을 나열하지 않습니다
- 절 제목 바로 아래에 **그 절의 질문이 무엇을 묻는지 한 줄**을 인용 블록으로 답니다
- **절 단위로 묶고**, 절 제목 옆에 `— [노트](<../N.N 절 제목.md>)` 링크를 답니다. 폴더 이름에 공백이 있으므로 **꺾쇠 링크(`](<...>)`)** 를 씁니다
- 번호는 **장 안에서 1부터 연속**입니다. 질문을 지우면 다시 매깁니다
- 문장 끝에 **중요도를 5점 척도(★)** 로 답니다. `★★★☆☆ 이상만` 수록합니다
  - ★★★★★ 반드시 답할 수 있어야 함 · ★★★★☆ 자주 나옴 · ★★★☆☆ 알아두면 좋음
- **노트에 답이 있는 것만** 질문으로 만듭니다. 노트에 없는 내용은 넣지 않습니다

**답변 파일**

- 제목은 `### <번호>. <짧은 주제> <별점>` 이고 **질문 번호와 1:1로 맞춥니다**
- 항목마다 같은 틀입니다 — **① 한 줄 답**(볼드) → **② ELI5 설명**(일상 비유 먼저, 용어는 그다음) → **③ 표 또는 mermaid** → **④ 덧붙일 것**(인용 블록)
- 정답 나열보다 **왜 그런지**와 **다른 장으로의 연결**에 무게를 둡니다
- mermaid는 **말로 하면 길어지는 것**에만 씁니다. 오류 상황(무한 루프, 충돌 순서)이 특히 효과가 좋습니다

**노트를 고치면 질문·답변도 함께 맞춥니다.** 특히 버전이나 동작이 바뀐 부분은 답변이 먼저 낡습니다.

> **질문 파일을 불리지 마세요.** 개념 표와 한 줄 안내까지가 선입니다. 설명·예제·노트 재서술이 들어가기 시작하면 노트·질문·답변 **세 곳을 동시에 관리**해야 합니다. 질문 파일은 **지도** 역할까지만 합니다.

## 4. 다이어그램 규칙

### ASCII 아트 금지, mermaid 사용

구조도·흐름도는 **mermaid**로 작성합니다. GitHub이 자동 렌더링하고, 텍스트라 diff 추적이 되며, 한글 폰트 문제가 없습니다.

- 순서·계층: `graph TD` / `graph LR` / `graph TB`
- 여러 주체가 주고받는 흐름: `sequenceDiagram` (예: 파드 삭제 절차, 배포 흐름)
- 줄바꿈은 `<br/>`, 노드 라벨은 큰따옴표로 감쌉니다

**예외**: 문자열 분해 주석(예: 파드 이름 `kiada-7d4c8f9b6d-x7k2m` 아래에 `+--`로 각 부분을 가리키는 것)은 고정폭 코드 블록이 더 읽히므로 그대로 둡니다.

### mermaid 문법 검증 (작성 후 필수)

렌더링 깨짐을 미리 잡습니다. Node.js 필요.

```powershell
# 스크래치 디렉터리에서
npm install mermaid@11 jsdom --no-audit --no-fund
node validate.mjs
```

> **이 PC에는 Node.js가 설치되어 있지 않습니다(2026-08-22 확인).** podman이 있으니 컨테이너로 돌립니다. Git Bash에서는 `MSYS_NO_PATHCONV=1`을 붙여야 `-w /work`가 윈도우 경로로 변환되지 않습니다.
>
> ```bash
> MSYS_NO_PATHCONV=1 podman run --rm \
>   -v "C:/Users/dorim/Study/Kubernetes-in-Action:/repo:ro" -v "$SCRATCH:/work" -w /work \
>   docker.io/library/node:22-alpine \
>   sh -c "npm install mermaid@11 jsdom --no-audit --no-fund --silent >/dev/null 2>&1 && node validate.mjs /repo"
> ```

`validate.mjs` 요지 — jsdom으로 DOM을 만든 뒤 모든 `.md`의 ` ```mermaid ` 블록을 `mermaid.parse()`에 통과시킵니다.

```js
import { JSDOM } from 'jsdom';
const dom = new JSDOM('<!doctype html><html><body></body></html>', { pretendToBeVisual: true });
global.window = dom.window; global.document = dom.window.document;
Object.defineProperty(global, 'navigator', { value: dom.window.navigator, configurable: true });
global.Element = dom.window.Element; global.HTMLElement = dom.window.HTMLElement;
global.SVGElement = dom.window.SVGElement; global.Node = dom.window.Node;
global.getComputedStyle = dom.window.getComputedStyle;
// 이후 mermaid 동적 import → mermaid.parse(code)
```

> `navigator`는 직접 대입하면 `TypeError`가 납니다. 반드시 `Object.defineProperty`를 씁니다.
> DOM 없이 실행하면 전부 `DOMPurify.addHook is not a function`으로 실패합니다. **문법 오류가 아니라 환경 문제**입니다.

## 5. 이미지 규칙

- 이미지는 **저장소 루트 `images/` 한 곳**에 모읍니다. 각 노트에서 `../images/파일명.svg`로 참조합니다
  - 장 폴더 이름에 한글·공백·마침표가 있어서, 공백이 섞인 상대 경로는 GitHub에서 깨집니다. `../images/`는 공백이 없어 안전합니다
- 출처는 **필수**입니다. 이미지 바로 아래에 캡션으로 답니다

```markdown
![설명](../images/파일명.svg)

> 출처: [문서 제목](원본 URL) © The Kubernetes Authors, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
```

- 새 이미지를 받으면 **`images/README.md` 표에 파일명·원본 URL·사용처를 추가**합니다
- 쿠버네티스 공식 문서는 CC BY 4.0이라 출처를 밝히면 사용·재배포가 가능합니다

## 6. 실습 환경

Windows 11 + **Podman + kind** 조합입니다. 구축 절차와 배경(왜 kind인가, 왜 노드 1개인가, 왜 Podman인가)은 [SETUP.md](SETUP.md)에 있습니다. **모든 장의 노트는 SETUP.md대로 환경을 갖춘 상태를 전제로 씁니다.**

### 설치된 것

| 도구 | 버전 |
| :--- | :--- |
| podman | 5.8.3 (machine: `podman-machine-default`, wsl, 4 CPU · 6GiB · 60GiB, **rootful**) |
| kind | 0.32.0 |
| kubectl | v1.36.3 |
| 클러스터 | `kia` — 단일 노드 `kia-control-plane`, 쿠버네티스 v1.36.1, containerd 2.3.1 |

- 사용자 환경변수 **`KIND_EXPERIMENTAL_PROVIDER=podman`** 이 영구 등록되어 있습니다
- kubectl 컨텍스트는 **`kind-kia`**

### 실습 시작하기 (PC를 껐다 켠 뒤 포함)

**PC를 끄면 podman machine이 함께 내려가고, kind 클러스터는 되살아나지 않습니다.** 재부팅 후에는 클러스터를 다시 만드는 것이 확실합니다.

```powershell
podman machine start
cd "C:\Users\dorim\Study\Kubernetes-in-Action"
kind delete cluster --name kia          # Exited 상태로 남은 것 정리
kind create cluster --config kind-config.yaml
kubectl get nodes
```

> **왜 재생성해야 하나 (2026-08-20 실측)**
> kind 노드 컨테이너는 재시작 정책이 `no`라서 machine을 켜도 `Exited`로 남습니다. `podman start`로 켜면 노드 안의 컨트롤 플레인은 정상 복구되지만(`crictl ps`로 확인됨), Windows의 kubectl이 API 서버에 붙지 못합니다 — `context deadline exceeded ... EOF`. VM 재시작 시 포트 포워딩 경로가 끊어지기 때문이며 `podman restart`로도 복구되지 않습니다. **재생성이 유일하게 확실한 방법이고 1분이면 됩니다.**

따라서 **실습 리소스를 클러스터 안에만 두면 안 됩니다.** 항상 YAML로 저장해 두고 다시 `kubectl apply` 하는 흐름을 전제로 노트를 씁니다.

### 단일 노드인 이유와 노드를 늘리는 법

- 학습에는 **노드 1개로 충분**합니다. 컨트롤 플레인 3대 권장은 etcd 과반수 합의 때문이며 **운영 환경 기준**입니다
- 단일 노드 kind 클러스터에는 **컨트롤 플레인 테인트가 없어서** 일반 파드가 그대로 배치됩니다
- **14장(레플리카셋), 17장(데몬셋)** 에서 파드가 여러 노드에 흩어지는 걸 보려면 `kind-config.yaml`에 `- role: worker`를 추가하고 클러스터를 재생성합니다. 워커를 추가하면 컨트롤 플레인 테인트가 되살아납니다
- `kind-config.yaml`에는 **12~13장 인그레스 실습용 80/443 포트 매핑**이 이미 들어 있습니다

### 자주 걸리는 문제

| 증상 | 해결 |
| :--- | :--- |
| `kubectl` 연결 거부 | `podman machine start` |
| `failed to create cluster: ... docker` | `$env:KIND_EXPERIMENTAL_PROVIDER = "podman"` |
| 권한 / cgroup 오류 | machine이 rootful인지 확인: `podman info --format "{{.Host.Security.Rootless}}"` → `false` |
| 노드가 `NotReady` | machine 메모리 부족. `podman machine set --memory 8192` |

## 7. 작업할 때 유의할 점

- **한국어로 작성합니다.** 기술 용어는 한글 + 영문 병기
- 셸은 **PowerShell**이 기본입니다. Bash 도구도 쓸 수 있지만 문법이 다릅니다
- 대용량 한글 문서는 **Bash 히어독으로 쓰지 마세요.** 특수문자에서 깨집니다. Write 도구를 씁니다
- 명령 실행 결과를 문서에 넣을 때는 **실제로 실행해서** 얻습니다
- 원서 목차나 쿠버네티스 최신 동작이 불확실하면 **웹으로 확인 후** 작성합니다. 추측한 버전 번호를 사실처럼 쓰지 않습니다
- 작업 후 **mermaid 검증**과 **이미지 경로 확인**을 돌립니다

```powershell
# 이미지 경로 깨짐 확인
Get-ChildItem -Recurse -Filter *.md | Where-Object { $_.Directory.Name -ne 'images' } | ForEach-Object {
  $md = $_
  foreach ($m in [regex]::Matches((Get-Content $md.FullName -Raw -Encoding UTF8), '!\[[^\]]*\]\(([^)]+)\)')) {
    $full = Join-Path $md.Directory.FullName $m.Groups[1].Value
    if (-not (Test-Path $full)) { "MISSING: {0} -> {1}" -f $md.Name, $m.Groups[1].Value }
  }
}
```

## 8. 참고 링크

- 저자 공식 예제 저장소: <https://github.com/luksa/kubernetes-in-action-2nd-edition>
- 원서 온라인(livebook): <https://livebook.manning.com/book/kubernetes-in-action-second-edition/>
- 쿠버네티스 공식 문서: <https://kubernetes.io/docs/>
- kind 문서: <https://kind.sigs.k8s.io/>
