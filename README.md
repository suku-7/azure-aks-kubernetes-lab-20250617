# Model
## azure-aks-kubernetes-lab-20250617
이 문서는 Azure Kubernetes Service (AKS) 클러스터 환경에서 Spring Boot 기반 애플리케이션을 배포하고 관리하는 일련의 과정을 담고 있습니다.  
Gitpod을 개발 환경으로 활용하여 AKS 클러스터 설정부터 애플리케이션 배포, 업데이트, 스케일링, 로드 밸런싱 및 YAML 파일을 통한 선언적 관리를 실습했습니다.  

## 사전 준비
Azure 계정 및 구독, Gitpod 워크스페이스, monolith-order Spring Boot 애플리케이션 코드

![스크린샷 2025-06-17 100224](https://github.com/user-attachments/assets/688aa955-f530-45e3-9d65-510f1eac29df)
![스크린샷 2025-06-17 100321](https://github.com/user-attachments/assets/46bf14c5-c108-47e4-9a39-9a8fa4ac6cd4)
![스크린샷 2025-06-17 100427](https://github.com/user-attachments/assets/9c7d64ce-8646-437f-913f-e377eba3ab7e)
![스크린샷 2025-06-17 111456](https://github.com/user-attachments/assets/5b199902-182c-4101-939c-5cd1669d1020)
![스크린샷 2025-06-17 112651](https://github.com/user-attachments/assets/d7bcb5c3-a729-4396-a7ec-aaba95c1a952)
![스크린샷 2025-06-17 114632](https://github.com/user-attachments/assets/c7885cdb-2019-4e65-9704-a88eb8cbf572)
![스크린샷 2025-06-17 115318](https://github.com/user-attachments/assets/b8e84dc8-f62b-4631-8a52-9e4526a5b20c)
![스크린샷 2025-06-17 120739](https://github.com/user-attachments/assets/a905c2a6-0cc2-43e6-9a37-4ee12f8d9525)
![스크린샷 2025-06-17 121047](https://github.com/user-attachments/assets/5fecbc08-b7dd-49d7-ac01-4e1a5a2d0020)
![스크린샷 2025-06-17 122016](https://github.com/user-attachments/assets/e6c19601-f01b-485f-b5b3-f785649e46a9)
![스크린샷 2025-06-17 122803](https://github.com/user-attachments/assets/a5ec3c6b-aacc-41f5-b2ab-6b0ba6d52afd)

## 실습 단계
1. Azure 환경 초기 설정
- 최초 Gitpod 워크스페이스 재시작 시 환경을 초기화하고 Java SDK를 설치합니다.
```
# Gitpod 환경 초기화 (필요시)
./init.sh

# Java SDK 설치 (필요시)
sdk install java
```
2. Azure 리소스 생성 (Azure Portal 또는 Azure CLI)
- Azure Portal에서 리소스 그룹과 AKS 클러스터를 미리 생성합니다.
- 리소스 그룹 생성: a071098-rsrcgrp
- Kubernetes 클러스터 생성: a071098-aks (메모리, 노드 수 등 설정)

3. Gitpod에서 Azure CLI 및 Kubectl 확인
- Gitpod 터미널에서 Azure CLI (az)와 Kubectl (kubectl)이 설치되었는지 확인합니다.
```
# Azure CLI 버전 확인
az --version

# Kubectl 버전 확인
kubectl version --client
```
4. Azure CLI 로그인 및 AKS 클러스터 연동
- Azure 계정으로 로그인하고, Gitpod에서 AKS 클러스터에 접근할 수 있도록 자격 증명을 설정합니다.
```
# Azure 계정 로그인 (브라우저를 통해 인증 코드 입력)
az login --use-device-code

# 여러 구독이 있을 경우, 사용할 구독 선택 (프롬프트에 숫자 '1' 입력 등)

# AKS 클러스터 자격 증명 가져오기 및 Kubectl 컨텍스트 설정
az aks get-credentials --resource-group a071098-rsrcgrp --name a071098-aks

# Kubectl이 클러스터에 정상 연결되었는지 확인
kubectl get all
kubectl get node
```
5. 주문 서비스 (Order Service) 배포 (명령어 방식)
- jinyoung/monolith-order:v202105042 이미지를 사용하여 order Deployment를 생성합니다.
```
# 'order' Deployment 생성
kubectl create deploy order --image=jinyoung/monolith-order:v202105042

# 모든 Kubernetes 리소스 상태 확인
kubectl get all
```
참고:  
- Deployment는 원하는 수의 파드를 생성하고 유지하는 상위 컨트롤러입니다.
- Deployment 생성 시 자동으로 ReplicaSet이 생성됩니다.
- ReplicaSet은 실제 서버 단위인 Pod를 복제하고 관리합니다.
- Deployment는 ReplicaSet을 통해 Pod들을 관리하며 항상 지정된 개수(Desired)를 유지합니다.

6. 배포된 파드 트러블슈팅 및 로그 확인
- 파드가 Error 또는 CrashLoopBackOff 상태일 경우, 상세 정보와 로그를 확인하여 문제 원인을 파악합니다.
```
# 파드 상태 확인 (예시: order-86b7fd4f57-sdzq8는 실제 파드 이름으로 교체)
kubectl get pod

# 특정 파드의 상세 정보 확인
kubectl describe po <pod-name>

# 특정 파드의 실시간 로그 확인
kubectl logs -f <pod-name>

# 특정 파드 내부 쉘 접속
kubectl exec -it <pod-name> -- /bin/sh
```
7. 애플리케이션 업데이트 (롤링 업데이트)
- 새로운 버전의 이미지(v20210504)로 order 서비스를 업데이트합니다. Kubernetes Deployment는 롤링 업데이트 방식으로 무중단 배포를 수행합니다.
```
# 'order' Deployment의 이미지 업데이트
kubectl set image deploy order monolith-order=jinyoung/monolith-order:v20210504

# 업데이트 진행 상황 및 리소스 상태 확인
kubectl get all

# (선택 사항) 이전 버전의 파드가 아직 남아있다면 수동으로 삭제하여 업데이트 촉진 (Deployment가 새 파드 생성)
# kubectl delete pod <old-pod-name>
```
- 참고: kubectl set image 명령은 Deployment의 롤링 업데이트 기능을 활용하여, 기존 ReplicaSet과 새로운 ReplicaSet을 통해 파드를 점진적으로 교체함으로써 무중단으로 애플리케이션 버전을 업데이트합니다. Deployment는 항상 'Desired' 파드 개수를 유지하며, 파드가 삭제되면 자동으로 재생성하여 서비스의 안정성을 보장합니다.

8. 파드 항상성 유지 테스트 (Self-healing)
- Deployment가 파드를 자동으로 복구하는 기능을 확인합니다.
```
# 터미널 1: 파드 목록을 지속적으로 감시
watch kubectl get pod

# 터미널 2: 'order' 앱 레이블을 가진 모든 파드 삭제
kubectl delete pod -l app=order
```
- 결과 확인: watch 터미널에서 파드가 삭제되자마자 새로운 파드가 생성되고 AGE가 초기화되는 것을 확인할 수 있습니다. 이는 Deployment의 자체 복구(Self-healing) 기능입니다.

9. 클러스터 외부 접근 설정
- order 서비스를 외부에서 접근 가능하도록 LoadBalancer 타입의 Service를 노출합니다.
```
# 'order' Deployment를 LoadBalancer 타입으로 노출 (8080 포트)
kubectl expose deploy order --type=LoadBalancer --port=8080

# LoadBalancer의 External IP 주소 확인 (Pending -> IP 할당까지 시간 소요)
kubectl get service -w
```
- 접속 확인: 할당된 External IP 주소와 포트를 사용하여 웹 브라우저(http://<EXTERNAL-IP>:8080/) 또는 터미널(http <EXTERNAL-IP>:8080)에서 서비스에 접속하여 정상 동작을 확인합니다.

10. 포트 포워딩을 통한 임시 통신
- LoadBalancer 없이 개발/테스트 목적으로 특정 파드에 임시적으로 접근하는 방법입니다.
```
# 터미널 1: 로컬 8080 포트를 'order' Deployment의 8080 포트로 포워딩
kubectl port-forward deploy/order 8080:8080

# 터미널 2: 로컬에서 서비스에 접속
curl localhost:8080
```
- 참고: 이는 개발/테스트용 임시 접근 방식이며, 실제 제품 환경에서는 로드 밸런서 사용이 권장됩니다.

11. 주문 서비스 인스턴스 확장 (수동 스케일링)
- order 서비스의 파드 개수를 수동으로 3개로 확장합니다.
```
# 'order' Deployment의 파드 개수를 3개로 설정
kubectl scale deploy order --replicas=3

# 파드 개수 확인
kubectl get pod
```
12. 주문 서비스 롤백
- 이전 버전의 이미지(v202105042)로 order 서비스를 롤백합니다.
```
# 'order' Deployment를 이전 버전으로 롤백
kubectl rollout undo deploy order

# 롤백 결과 및 이미지 버전 확인
kubectl get deploy -o wide
kubectl get all
```
13. 리소스 삭제
- 배포된 Deployment와 Service를 깔끔하게 삭제합니다.
```
# 'order' Deployment 삭제 (연관된 ReplicaSet 및 Pod도 함께 삭제됨)
kubectl delete deploy order

# 'order' Service 삭제
kubectl delete service order

# 모든 리소스가 삭제되었는지 확인
kubectl get all
```
14. YAML 파일을 이용한 선언적 배포 (추가 실습)
- 명령어 방식 대신 YAML 파일을 사용하여 리소스 스펙을 정의하고 배포합니다.
```
# Lab 디렉토리로 이동 (order.yaml 파일이 있는 곳)
cd Lab/

# order.yaml 파일의 내용을 확인 (Deployment, ReplicaSet, Pod 스펙 정의)
# cat order.yaml

# YAML 파일 적용
kubectl apply -f order.yaml

# 배포된 리소스 확인
kubectl get all

# (선택 사항) YAML 파일을 통해 배포된 서비스에 포트 포워딩
kubectl port-forward deploy/order-by-yaml 8080:8080

# (선택 사항) 현재 배포된 Deployment의 YAML 스펙을 파일로 추출 (수정 후 재적용 가능)
kubectl get deploy order-by-yaml -o yaml > order2.yaml
```
