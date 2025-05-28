# snap-it-word2vec-manifest 저장소 구조

현재 CI 코드에 맞게 생성된 snap-it-word2vec-manifest 저장소의 구조는 다음과 같습니다:

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

## CI/CD 연동 동작 방식

1. 소스 코드가 `main` 브랜치에 푸시되면 GitHub Actions 워크플로우가 실행됩니다.
2. 워크플로우는 Docker 이미지를 빌드하고 GitHub Container Registry에 푸시합니다.
3. 그런 다음 `snap-it-word2vec-manifest` 저장소를 체크아웃하고 `values.yaml` 파일의 이미지 태그를 새 commit SHA로 업데이트합니다.
4. 변경 사항은 매니페스트 저장소에 커밋됩니다.
5. GitOps 도구가 이 저장소를 모니터링하여 변경 사항을 감지하고 클러스터에 적용합니다.

## 주요 설정 사항

- **네임스페이스**: `snapit-word2voc`
- **서비스 타입**: NodePort
- **외부 접근 포트**: 32766
- **이미지 저장소**: ghcr.io/chabinhwang/snap-it-word2vec
- **리소스 제한**:
  - CPU 제한: 500m
  - 메모리 제한: 512Mi
  - CPU 요청: 200m
  - 메모리 요청: 256Mi

이 구조는 현재 CI 파이프라인 코드와 호환되며, 이미지 태그가 업데이트될 때 자동으로 배포를 트리거할 수 있습니다. 