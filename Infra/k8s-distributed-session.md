# Kubernetes 핵심 개념 (프론트엔드 개발자 기준)

분산 환경에서 세션 문제를 겪으면서 K8s 개념을 실제로 체감하게 됨. 인프라 맵을 시각화하는 작업을 하면서 Pod·Deployment·Service·Namespace가 코드 레벨에서 어떻게 연결되는지 정리함.

---

## 1. Pod

실행 중인 컨테이너의 단위. 쉽게 말하면 실제로 돌아가는 프로세스 하나임.

- `phase`가 Running이면 정상, 아니면 문제 있는 상태
- `restartCount`는 Pod가 죽었다 살아난 횟수
- Pod는 죽었다 살아나면 **IP가 바뀜** → 이게 Service가 필요한 이유

---

## 2. Deployment

Pod를 몇 개 띄울지 관리하는 단위. "이 Pod 항상 3개 유지해라" 같은 설정을 담당함.

- `replicas`: 목표 Pod 수
- `readyReplicas`: 실제로 준비된 Pod 수
- 둘이 다르면 뭔가 문제 있는 상태

인프라 맵 UI 예: `subtitle: ${deployment.readyReplicas}/${deployment.replicas} replicas`

---

## 3. Service

Pod IP는 재시작마다 바뀌기 때문에, 고정 엔드포인트 역할을 하는 게 Service임. "이 라벨 가진 Pod들한테 트래픽 보내줘" 하는 방식으로 동작함.

- Service의 `selector`와 Pod의 `labels`가 매칭되면 연결됨
- 인프라 맵에서 Service → Pod 연결선이 이 로직으로 그려짐

```typescript
const linkedBySelector = matchesSelector(pod.labels, service.selector)
```

---

## 4. Namespace

리소스를 논리적으로 격리하는 그룹. 같은 클러스터 안에서 prod, dev, monitoring 식으로 나눌 수 있음.

- 다른 Namespace의 리소스끼리는 기본적으로 격리됨
- 인프라 맵에서 연결선 하이라이트를 Namespace 기준으로 제한하는 이유가 여기 있음

---

## 5. 분산 환경 세션 문제와의 연결

Deployment의 `replicas: 3`이면 Pod가 3개 뜸. 유저 요청이 매번 다른 Pod로 라우팅될 수 있음.

- Pod A 메모리에 저장한 세션을 Pod B는 모름 → 로그인이 풀리는 현상 발생
- 해결: 세션을 서버 메모리가 아닌 AES-256-GCM으로 암호화한 쿠키에 저장
- 어떤 Pod가 요청을 받아도 쿠키만 복호화하면 세션 복원 가능

| 방식 | 문제 |
|------|------|
| 서버 메모리 세션 | Pod마다 세션 공유 안 됨 |
| 암호화 쿠키 (stateless) | 어떤 Pod든 복호화 가능 → 문제 없음 |
