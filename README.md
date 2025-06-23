# Snap-It Word2Vec Kubernetes 매니페스트

snap-it-word2vec 애플리케이션을 Kubernetes 클러스터에 배포하기 위한 Helm Chart 리포지토리입니다.

## 📁 프로젝트 구조

```
snap-it-word2vec-manifest/
├── README.md                    # 이 문서
├── Chart.yaml                   # Helm Chart 메타데이터
├── values.yaml                  # 기본 설정 값
├── snap-it-manifests-structure.md  # 매니페스트 구조 설명
└── templates/
    ├── deployment.yaml          # 애플리케이션 배포 설정
    ├── service.yaml             # 서비스 노출 설정
    ├── pv.yaml                  # Persistent Volume 설정
    ├── hpa.yaml                 # Horizontal Pod Autoscaler
    └── NOTES.txt                # 배포 후 안내 메시지
```

## ⚙️ 주요 설정 정보

### 기본 설정 (values.yaml)

| 항목 | 값 | 설명 |
|------|-----|------|
| **네임스페이스** | `snapit-word2voc` | 애플리케이션 배포 네임스페이스 |
| **복제본 수** | `2` | 고가용성을 위한 파드 복제본 |
| **이미지** | `ghcr.io/chabinhwang/snap-it-word2vec` | 컨테이너 이미지 저장소 |
| **서비스 타입** | `NodePort` | 외부 접근을 위한 서비스 타입 |
| **외부 포트** | `32765` | NodePort를 통한 외부 접근 포트 |
| **내부 포트** | `8000` | 컨테이너 내 애플리케이션 포트 |

### 리소스 할당

| 구분 | CPU | Memory | 설명 |
|------|-----|---------|------|
| **Requests** | 500m | 4096Mi | 최소 보장 리소스 |
| **Limits** | 1500m | 5120Mi | 최대 사용 가능 리소스 |

### 자동 확장 설정 (HPA)

- **활성화**: 현재 비활성화 (`enabled: false`)
- **최소 복제본**: 1개
- **최대 복제본**: 2개  
- **CPU 임계값**: 90%

## 🚀 배포 방법

### 1. 사전 준비

#### 모델 파일 준비
```bash
# 호스트 시스템에 모델 디렉터리 생성
sudo mkdir -p /mnt/data/models

# Google News Word2Vec 모델 다운로드 (약 1.5GB)
sudo wget -O /mnt/data/models/GoogleNews-vectors-negative300.bin.gz \
  https://s3.amazonaws.com/dl4j-distribution/GoogleNews-vectors-negative300.bin.gz

# 적절한 권한 설정
sudo chmod 644 /mnt/data/models/GoogleNews-vectors-negative300.bin.gz
```

#### 네임스페이스 생성
```bash
kubectl create namespace snapit-word2voc
```

### 2. Helm 배포

```bash
# 차트 디렉터리로 이동
cd snap-it-word2vec-manifest

# Helm을 사용한 배포
helm install snap-it-word2vec . -n snapit-word2voc

# 또는 업그레이드
helm upgrade snap-it-word2vec . -n snapit-word2voc
```

### 3. 수동 배포 (kubectl)

```bash
# Persistent Volume 생성
kubectl apply -f templates/pv.yaml

# 애플리케이션 배포
kubectl apply -f templates/deployment.yaml

# 서비스 생성
kubectl apply -f templates/service.yaml

# HPA 적용 (필요 시)
kubectl apply -f templates/hpa.yaml
```

## 🔍 배포 확인 및 모니터링

### 배포 상태 확인

```bash
# 파드 상태 확인
kubectl get pods -n snapit-word2voc

# 서비스 확인
kubectl get svc -n snapit-word2voc

# PV/PVC 상태 확인
kubectl get pv,pvc -n snapit-word2voc

# 리소스 사용량 확인
kubectl top pods -n snapit-word2voc
```

### 로그 및 디버깅

```bash
# 애플리케이션 로그 확인
kubectl logs -f deployment/snap-it-word2vec -n snapit-word2voc

# 파드 이벤트 확인
kubectl get events -n snapit-word2voc --sort-by='.lastTimestamp'

# 파드 내부 접근
kubectl exec -it <pod-name> -n snapit-word2voc -- bash
```

### 헬스 체크

```bash
# 클러스터 내에서 헬스 체크
kubectl run test-pod --image=curlimages/curl:latest --rm -it --restart=Never \
  -- curl http://snap-it-word2vec.snapit-word2voc.svc.cluster.local:8000/health

# NodePort를 통한 외부 접근 테스트
curl http://<NODE-IP>:32765/health
```

## 🔧 설정 커스터마이징

### values.yaml 수정

주요 설정을 변경하려면 `values.yaml` 파일을 수정하거나 배포 시 값을 오버라이드하세요:

```bash
# 복제본 수 변경
helm upgrade snap-it-word2vec . -n snapit-word2voc --set replicaCount=3

# 리소스 제한 변경
helm upgrade snap-it-word2vec . -n snapit-word2voc \
  --set resources.limits.memory=6Gi \
  --set resources.limits.cpu=2000m

# 외부 포트 변경
helm upgrade snap-it-word2vec . -n snapit-word2voc --set service.nodePort=32766
```

### 환경별 설정

```bash
# 개발 환경용 values 파일 생성
cp values.yaml values-dev.yaml

# 개발 환경 배포
helm install snap-it-word2vec . -n snapit-word2voc -f values-dev.yaml
```

## 🔄 CI/CD 통합

### GitHub Actions 워크플로우

이 매니페스트는 [snap-it-word2vec](https://github.com/chabinhwang/snap-it-word2vec) 저장소의 CI/CD 파이프라인과 연동됩니다:

1. **빌드 트리거**: `main` 브랜치에 푸시 시
2. **이미지 빌드**: Docker 이미지를 GitHub Container Registry에 푸시
3. **자동 업데이트**: `values.yaml`의 이미지 태그 자동 업데이트
4. **배포**: Kubernetes 클러스터에 자동 배포 (설정 시)

### 수동 이미지 업데이트

```bash
# 새 이미지 태그로 업데이트
helm upgrade snap-it-word2vec . -n snapit-word2voc \
  --set image.tag=<new-commit-hash>
```

## 🛠️ 문제 해결

### 일반적인 문제

1. **파드가 시작되지 않는 경우**
   ```bash
   # 파드 상세 정보 확인
   kubectl describe pod <pod-name> -n snapit-word2voc
   
   # 이벤트 확인
   kubectl get events -n snapit-word2voc
   ```

2. **모델 파일 로딩 실패**
   ```bash
   # PV 마운트 확인
   kubectl exec -it <pod-name> -n snapit-word2voc -- ls -la /app/models/
   
   # 호스트 파일 확인
   ls -la /mnt/data/models/
   ```

3. **메모리 부족 오류**
   ```bash
   # 리소스 제한 증가
   helm upgrade snap-it-word2vec . -n snapit-word2voc \
     --set resources.limits.memory=6Gi
   ```

### 성능 최적화

- **읽기 전용 모드**: 모델 파일은 읽기 전용으로 마운트
- **리소스 요청**: 적절한 CPU/메모리 요청으로 스케줄링 최적화
- **프로브 설정**: 모델 로딩 시간을 고려한 헬스 체크 타이밍

## 📚 참고 문서

- [Helm 공식 문서](https://helm.sh/docs/)
- [Kubernetes 리소스 관리](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [애플리케이션 저장소](https://github.com/chabinhwang/snap-it-word2vec)

## 🤝 기여하기

매니페스트 개선이나 새로운 기능 추가는 언제든 환영합니다:

1. Issue 생성으로 문제점이나 개선사항 공유
2. Fork & Pull Request로 변경사항 제안
3. 배포 테스트 및 피드백 제공

## 📄 라이센스

이 프로젝트는 MIT 라이센스 하에 배포됩니다.
