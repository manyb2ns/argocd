# ArgoCD ApplicationSet 예시

이 디렉토리에는 환경별 브랜치를 구분하여 ApplicationSet을 구성하는 최소한의 예시가 포함되어 있습니다.

**ApplicationSet은 DEV와 PRD 환경으로 분리되어 구성되어 있으며, 환경별로 다른 Git 브랜치를 사용합니다.**

## 디렉토리 구조

```
application_sets/
├── git-directory-applicationset.yaml  # ApplicationSet 정의
├── templates/                          # 애플리케이션 템플릿 파일
│   ├── cluster1/
│   │   ├── nginx-deploy.yaml
│   │   └── nginx-config.yaml
│   └── cluster2/
│       ├── nginx-deploy.yaml
│       ├── index.html
│       └── kustomization.yaml
└── README.md
```

### 디렉토리 구조에 대한 권장사항

**⚠️ 주의: ApplicationSet 정의와 템플릿 파일을 같은 디렉토리에 두는 것에 대한 고려사항**

현재 구조는 ApplicationSet 정의(`git-directory-applicationset.yaml`)와 실제 배포할 템플릿 파일(`templates/`)을 같은 디렉토리(`application_sets/`) 내에 하위 디렉토리로 분리하여 구성했습니다.

**이 방식의 장점:**
- ✅ 관련 파일들이 한 곳에 모여 있어 관리가 편리합니다
- ✅ 작은 프로젝트나 학습 목적에는 충분히 실용적입니다
- ✅ 하위 디렉토리로 분리되어 있어 구조가 명확합니다

**더 권장되는 대안 구조:**
- **옵션 1**: ApplicationSet 정의와 템플릿을 완전히 분리
  ```
  02_applications/
  ├── application_sets/          # ApplicationSet 정의만
  │   └── git-directory-applicationset.yaml
  └── templates/                 # 템플릿 파일만
      ├── cluster1/
      └── cluster2/
  ```
  
- **옵션 2**: GitOps 리포지토리 구조에 맞춰 분리
  ```
  argocd/
  ├── applicationsets/           # ApplicationSet 정의
  └── apps/                      # 애플리케이션 템플릿
      ├── cluster1/
      └── cluster2/
  ```

**현재 구조를 사용해도 되는 경우:**
- 프로젝트 규모가 작고 관리가 단순한 경우
- 학습이나 프로토타이핑 목적
- 팀 내에서 구조에 대한 합의가 있는 경우

**대규모 프로젝트에서는 권장하지 않는 이유:**
- ApplicationSet 정의와 템플릿이 혼재되어 확장성이 떨어질 수 있음
- 권한 관리가 복잡해질 수 있음 (ApplicationSet은 관리자, 템플릿은 개발자)
- CI/CD 파이프라인 구성이 복잡해질 수 있음

현재 구조는 작은 프로젝트나 학습 목적에는 적합하지만, 프로덕션 환경이나 대규모 프로젝트로 확장할 때는 위의 대안 구조를 고려하는 것을 권장합니다.

## 환경 구조

- **DEV 환경**: 개발/테스트 환경 (`dev` 네임스페이스, `develop` 브랜치)
- **PRD 환경**: 프로덕션 환경 (`prd` 네임스페이스, `main` 브랜치)

각 환경은 독립적인 네임스페이스와 Git 브랜치에서 실행되며, 애플리케이션 이름과 레이블로 구분됩니다.

### 브랜치 기반 환경 분리

**환경별 브랜치 분리는 권장되는 접근 방식입니다:**
- ✅ DEV 환경에서 코드를 수정해도 PRD 환경에 영향을 주지 않음
- ✅ 각 환경별로 독립적인 배포 파이프라인 운영 가능
- ✅ PRD 환경은 안정적인 `main` 브랜치만 사용
- ✅ DEV 환경은 `develop` 브랜치에서 자유롭게 테스트 가능

## 파일 설명

### git-directory-applicationset.yaml
**Matrix Generator**를 사용하여 Git 리포지토리의 특정 디렉토리와 환경 조합으로 자동으로 애플리케이션을 생성하는 예시입니다.

**주요 특징:**
- Git 리포지토리를 스캔하여 지정된 경로 패턴에 맞는 디렉토리를 자동으로 찾습니다
- 각 디렉토리마다 DEV와 PRD 환경에 대해 자동으로 Application을 생성합니다
- 경로에서 클러스터 이름을 추출하여 해당 클러스터에 배포합니다
- **Git Directory Generator와 List Generator를 Matrix로 조합하여 환경별 애플리케이션을 생성합니다**
- **환경별로 다른 브랜치를 사용합니다 (DEV: `develop`, PRD: `main`)**
- **템플릿 파일 경로**: `02_applications/application_sets/templates/cluster*/*`

**생성되는 애플리케이션 예시:**
- `nginx-deploy-cluster1-dev` → `kind-cluster1` / `dev` 네임스페이스 / `develop` 브랜치
- `nginx-deploy-cluster1-prd` → `kind-cluster1` / `prd` 네임스페이스 / `main` 브랜치
- `nginx-deploy-cluster2-dev` → `kind-cluster2` / `dev` 네임스페이스 / `develop` 브랜치
- `nginx-deploy-cluster2-prd` → `kind-cluster2` / `prd` 네임스페이스 / `main` 브랜치

**사용 방법:**
```bash
kubectl apply -f git-directory-applicationset.yaml
```

## 사용된 Generator 타입

### Matrix Generator
- 여러 Generator를 조합하여 모든 조합에 대해 애플리케이션을 생성합니다.
- 이 예시에서는 Git Directory Generator와 List Generator를 조합하여 디렉토리별 × 환경별 애플리케이션을 생성합니다.

### Git Directory Generator
- Git 리포지토리를 스캔하여 지정된 경로 패턴에 맞는 디렉토리를 자동으로 찾습니다.
- 새 디렉토리가 추가되면 자동으로 애플리케이션이 생성됩니다.

### List Generator
- 정적으로 정의된 리스트를 기반으로 애플리케이션을 생성합니다.
- 이 예시에서는 환경 목록(DEV, PRD)을 정의하여 각 환경별로 애플리케이션을 생성합니다.

## 환경 분리 구조

모든 ApplicationSet은 다음과 같이 환경을 분리합니다:

### Git 브랜치
- **DEV**: `develop` 브랜치 사용
- **PRD**: `main` 브랜치 사용
- 각 환경은 독립적인 브랜치에서 코드를 관리하므로, 한 환경의 변경사항이 다른 환경에 영향을 주지 않습니다.

### 네임스페이스
- **DEV**: `dev` 네임스페이스
- **PRD**: `prd` 네임스페이스

### 애플리케이션 이름
- 환경 정보가 애플리케이션 이름에 포함됩니다 (예: `nginx-cluster1-dev`, `nginx-cluster1-prd`)

### 레이블
- 모든 애플리케이션에 `environment` 레이블이 추가됩니다
- 클러스터 정보는 `cluster` 레이블로 관리됩니다

## 주의사항

1. **네임스페이스**: ApplicationSet 리소스는 `argocd` 네임스페이스에 생성되어야 합니다.
2. **프로젝트**: `project` 필드에 유효한 ArgoCD 프로젝트를 지정해야 합니다.
3. **클러스터 URL**: 실제 클러스터의 API 서버 URL을 사용해야 합니다. 예시에서는 `https://kubernetes.default.svc`를 사용했지만, 실제 환경에 맞게 수정해야 합니다.
4. **Git 리포지토리**: 실제 사용하는 Git 리포지토리 URL로 변경해야 합니다.
5. **네임스페이스 생성**: `CreateNamespace=true` 옵션이 활성화되어 있어 자동으로 네임스페이스가 생성됩니다. 필요에 따라 `dev`와 `prd` 네임스페이스를 미리 생성할 수도 있습니다.
6. **Git 브랜치**: 환경별 브랜치 이름은 프로젝트에 맞게 조정할 수 있습니다. 기본값은 DEV: `develop`, PRD: `main`입니다.
7. **브랜치 존재 확인**: ApplicationSet을 적용하기 전에 해당 브랜치가 Git 리포지토리에 존재하는지 확인해야 합니다.

## 추가 리소스

- [ArgoCD ApplicationSet 공식 문서](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/)
- [ApplicationSet Generators 가이드](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators/)

