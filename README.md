<div align="center">

# SK076 · Helm Charts

**차트 작성부터 업그레이드·롤백, 저장소 게시와 재설치까지**

![Helm](https://img.shields.io/badge/Package-Helm-0F1689?style=flat-square)
![GitHub Pages](https://img.shields.io/badge/Repository-GitHub%20Pages-222222?style=flat-square)
![Chart](https://img.shields.io/badge/Published-0.0.1%20%7C%200.0.2-2563EB?style=flat-square)

[차트 인덱스](https://baekjiheon.github.io/mycharts/index.yaml) · [전체 실습](https://github.com/BaekJiHeon/myfirst-api-server) · [배포된 Helm 앱](https://sk076-ingress.class-3.skala-ai.com)

</div>

---

## 저장소 소개

Spring API를 Helm 차트로 패키징하고 GitHub Pages와 교육 파일 서버에 게시한 실습 저장소입니다. 설치 → 테스트 → Python 이미지로 업그레이드 → 롤백 → 삭제 → 저장소에서 재설치하는 흐름을 실제 실행했습니다.

**작성자:** BaekJiHeon · **실습 대상:** `class-3 / sk076` · **검증일:** 2026-10-27 KST

## 게시 버전

| 대상 | 버전 | 설명 |
|---|---|---|
| `index.yaml` 등록 패키지 | `0.0.1`, `0.0.2` | 다운로드·설치한 게시 버전 |
| 작업 소스 `Chart.yaml` | `0.1.0` | 현재 소스 버전, 아직 index에 게시하지 않음 |
| appVersion | `1.0` | 앱 버전 메타데이터 |

차트 버전, 앱 버전, 컨테이너 이미지 태그는 서로 다른 값입니다. 재현 설치는 게시된 `--version 0.0.2`를 명시합니다.

## 저장소 추가와 조회

```sh
helm repo add sk076 https://baekjiheon.github.io/mycharts
helm repo update sk076
helm search repo sk076/sk076-myfirst-helmchart --versions
helm show values sk076/sk076-myfirst-helmchart --version 0.0.2
```

교육 파일 서버에서도 같은 게시 버전을 제공합니다.

```sh
helm repo add sk076-course https://skala-fileserver.skala-ai.com/mycharts/sk076
helm repo update sk076-course
helm search repo sk076-course/sk076-myfirst-helmchart --versions
```

## 설치와 연결 테스트

아래 명령은 교육 클러스터 접근 권한이 있는 환경 기준입니다. 기존 `sk076-myfirst-helm` 릴리스가 있다면 새 설치 대신 `helm status`로 확인합니다.

```sh
helm --kube-context skala-admin-sa@class-3 install sk076-myfirst-helm   sk076/sk076-myfirst-helmchart --version 0.0.2   --namespace class-3 --wait --timeout 5m

helm --kube-context skala-admin-sa@class-3 test sk076-myfirst-helm   --namespace class-3 --logs
```

## 업그레이드와 롤백

실습에서는 이미지 `2.0`과 replica 2개로 변경한 뒤, 설치 직후 revision 1로 롤백했습니다. 기존 릴리스를 다룰 때는 `helm history`로 돌아갈 revision을 먼저 확인합니다.

```sh
helm --kube-context skala-admin-sa@class-3 upgrade sk076-myfirst-helm   sk076/sk076-myfirst-helmchart --version 0.0.2   --namespace class-3 --reuse-values   --set image.tag=2.0,replicaCount=2 --wait --timeout 5m

helm --kube-context skala-admin-sa@class-3 history sk076-myfirst-helm -n class-3
helm --kube-context skala-admin-sa@class-3 rollback sk076-myfirst-helm 1   --namespace class-3 --wait --timeout 5m
```

## 주요 values

다음 표는 현재 작업 소스의 [values.yaml](sk076-myfirst-helmchart/values.yaml) 기준입니다. 게시 패키지의 값은 `helm show values --version 0.0.2`로 확인합니다.

| 키 | 기본값 / 의미 |
|---|---|
| `replicaCount` | `1` |
| `image.repository` | `skala-registry.skala-ai.com/class-3/sk076-webserver` |
| `image.tag` | `1.0.docker` |
| `service.port` | `8080` |
| `fullnameOverride` | `sk076-myfirst-helmchart` |
| `ingress.enabled` | `true` |
| `ingress.className` | `public-nginx` |
| `ingress.hosts[0].host` | `sk076-ingress.class-3.skala-ai.com` |
| `autoscaling.enabled` | `false` |

교육용 namespace, Ingress와 TLS Secret을 전제로 합니다. 다른 클러스터에서는 해당 설정을 변경해야 합니다.

## 로컬 차트 검증

```sh
helm lint ./sk076-myfirst-helmchart
helm template sk076-myfirst-helm ./sk076-myfirst-helmchart -n class-3
```

## 실행 결과

| 실습 | 결과 |
|---|---|
| 차트 생성·lint·렌더링 | 통과 |
| install + helm test | 성공 |
| upgrade | Python 2.0 이미지, replica 2개 확인 |
| rollback | Spring 배포로 복귀 |
| uninstall + 저장소 재설치 | 성공 |
| GitHub Pages | 0.0.1 / 0.0.2 게시 |
| 교육 파일 서버 | 차트 2개 + index 업로드, 설치·테스트 성공 |
| 템플릿 | if/default/required/upper/toYaml/with/range/include 등 11개 검증 통과 |

상세 로그와 JSON은 [전체 실습의 evidence 폴더](https://github.com/BaekJiHeon/myfirst-api-server/tree/main/docs/evidence)에 있습니다. 해당 실습 저장소는 비공개이므로 접근 권한이 필요합니다. 검증 결과는 기록 시점의 스냅샷입니다.

## 파일 구성

```text
mycharts/
├── index.yaml
├── sk076-myfirst-helmchart-0.0.1.tgz
├── sk076-myfirst-helmchart-0.0.2.tgz
└── sk076-myfirst-helmchart/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
```

전체 소스 출처와 실습 맥락은 [DevOps Lab README](https://github.com/BaekJiHeon/myfirst-api-server#readme)에 정리했습니다.
