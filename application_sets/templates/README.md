# ApplicationSet 템플릿 디렉토리

이 디렉토리에는 ApplicationSet이 자동으로 스캔하여 배포할 애플리케이션 템플릿 파일들이 포함되어 있습니다.

## 디렉토리 구조

```
templates/
├── cluster1/
│   ├── nginx-deploy.yaml
│   └── nginx-config.yaml
└── cluster2/
    ├── nginx-deploy.yaml
    ├── index.html
    └── kustomization.yaml
```

## 구조 설명

- **cluster1/**: `kind-cluster1`에 배포될 애플리케이션 템플릿
- **cluster2/**: `kind-cluster2`에 배포될 애플리케이션 템플릿

각 클러스터 디렉토리 내의 파일들은 해당 클러스터에 배포될 Kubernetes 리소스 매니페스트입니다.

## 주의사항

- 이 디렉토리의 파일들은 Git Directory Generator에 의해 자동으로 스캔됩니다.
- 새 디렉토리나 파일을 추가하면 ApplicationSet이 자동으로 새로운 Application을 생성합니다.
- 파일 경로에서 클러스터 이름이 추출되어 배포 대상 클러스터로 사용됩니다.

