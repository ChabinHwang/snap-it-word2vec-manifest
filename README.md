# snap-it-word2vec-manifest

snap-it-word2vec 애플리케이션을 위한 Kubernetes 매니페스트 리포지토리입니다.

## 저장소 구조

```
snap-it-word2vec-manifest/
├── README.md
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── NOTES.txt
    └── service.yaml
```

## 주요 설정 정보

- **네임스페이스**: `snapit-word2voc`
- **서비스 타입**: NodePort
- **외부 접근 포트**: 32766
- **이미지 저장소**: `ghcr.io/chabinhwang/snap-it-word2vec`
- **리소스 제한**:
  - CPU: 500m (0.5 코어)
  - 메모리: 512Mi

## 배포 방법

### 수동 배포 (Helm)

```bash
# 네임스페이스 생성
kubectl create namespace snapit-word2voc

# 매니페스트 적용
helm install snap-it-word2vec . -n snapit-word2voc
```

### CI/CD 연동

CI/CD 파이프라인과 연동되어 있으며, `snap-it-word2vec` 저장소의 `main` 브랜치에 변경사항이 발생하면 자동으로 이미지 태그가 업데이트됩니다.

자세한 CI/CD 워크플로우 정보는 [snap-it-word2vec](https://github.com/chabinhwang/snap-it-word2vec) 저장소의 `.github/workflows/ci.yaml` 파일을 참조하세요.
