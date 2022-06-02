# Prometheus-practice
 모니터링 프로메테우스 연습
#### * Progress in macOS Monterey 12.3.1 and ubuntu 20.04
#### * Hardware is MacBook Pro (15-inch, 2016) and hp laptop

 # 필요한 소프트웨어
 ## 1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 6.0.4
: 오픈소스 하이퍼바이저. 가상 시스템을 원하는 이미지로 시작할 수 있음. 또한 가상 네트워크를 만들고 호스트의 파일 시스템 경로를 게스트에 마운트 가능하다.
 ## 2. [Vagrant](https://www.vagrantup.com/downloads) 2.2.4
: 가상 머신을 쉽게 만들고 관리해주는 툴이다.
 ## 3. [Minikube](https://minikube.sigs.k8s.io/docs/start/) 1.0.1
: 로컬 환경에서 쿠버네티스 환경을 구성하고 테스트할 수 있는 도구이다.
 ## 4. [kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-macos/) 1.14.1
: 쿠버네티스 클러스터를 제어하기 위한 커맨드 라인 도구이다.




<br/>

# 1. 환경설정 연습하기
#### 테스트 환경 구축
    $ vagrant up
#### vagrant running 확인
    $ vagrant status
#### vagrant 로그인
    $ vagrant ssh prometheus

</br>

#### 프로메테우스, 그라파나, 알림 메니저의 HTTP 엔드포인트
    프로메테우스 : 192.168.56.10:9090/targets
    그라파나 : 192.168.56.11:3000
    알림 매니저 : 192.168.56.12:9093

#### 테스트 환경 파괴
    $ vagrant destroy -f




2. 독립 실행형 서버에서 환경설정 적용 및 동시에 검증하기

#### 현재 환경설정 검증 (vagrant 로그인 이후에)
    $ cat /etc/systemd/system/prometheus.service
#### 프로메테우스 환경설정 파일 보기
    $ cat /etc/prometheus/prometheus.yml

#### 쿠버네티스 환경에서 프로메테우스 관리하기
##### 1. 미니쿠베 인스턴스가 실행 중이 아님을 확인하기
    $ minikube status
    $ minikube delete
##### 2.미니쿠베 인스턴스 실행
    $ minikube start \ --cpus=2 \ --memory=2048 \ --kubernetes-version=="v1.14.0" \ --vm-driver=virtualbox
    $ kubectl version --client (쿠버네티스 버전 확인)
##### 3. 쿠버네티스 대시보드 진입
    $ minikube dashboard
##### * 주의사항
<div align="center">
<img src="https://user-images.githubusercontent.com/54494793/170822639-e9013849-9e5b-4a39-a38d-b31c20a7993c.png" width="100%" height="100%" style="border-radius:5%;">
</div>
</br>

#### Exiting due to HOST_BROWSER: failed to open browser: exit status 1 

#### -> 맥의 기본 브라우저를 chrom에서 safari로 바꾸면 해결됨.

##### 4. manifest를 사용해 새로운 모니터링 네임스페이스를 생성함
    $ cd project2/provision/kubernetes/static
    # apiVersion : v1
    # kind: Namespace
    # metadata:
    #    name: monitoring
    $ kubectl apply -f monitoring-namespace.yaml

##### 5. kubernetes 대시보드에서 Namespace - monitoring 추가된 것 확인.


#### 쿠버네티스에서 프로메테우스 서버 배포


### (정적 환경설정)

##### 1. 간단한 환경설정해주기
    $ cd project2/provision/kubernetes/operator/
    $ kubectl apply -f prometheus-configmap.yaml

##### 2. 배포단계. 이전에 구성한 configmap을 pod에 마운트하기
    $ kubectl apply -f prometheus-deployment.yaml

##### * 배포 상태 확인하기
    $ kubectl rollout status deployment/prometheus-deployment -n monitoring

##### * 프로메테우스의 인스턴스의 로그 확인하기
    $ kubectl logs --tail=20 -n monitoring -l app=prometheus

##### 3. 별도의 포트포워딩 없이 접근하기
    $ kubectl apply -f prometheus-service.yaml

##### 4. 프로메테우스 웹 인터페이스에 연결하기
    $ minikube service prometheus-service -n monitoring


#### 쿠버네티스에서 프로메테우스 타깃 추가

##### 1. Hey 애플리케이션 배포
    $ kubectl apply -f hey-deployment.yaml

##### 2. 프로메테우스 웹 인터페이스에 연결하기
    $ kubectl rollout status deployment/hey-deployment -n monitoring
    $ kubectl logs --tail=20 -n monitoring -l app=hey
##### 3. 매니페스트 적용
    $ kubectl apply -f hey-service.yaml
##### 4. hey 웹 인터페이스에 연결하기.
    $ minikube service hey-service -n monitoring
##### 5. hey의 매트릭 수집을 위해 정적 타깃으로 추가해야 함.
    $ kubectl create -f prometheus-configmap-update.yaml -o yaml --dry-run | kubectl apply -f -
##### 6. 배포버전 어노테이션 적용.
    $ kubectl apply -f prometheus-deployment-update.yaml
##### 7. 배포 상태 확인
    $ kubectl rollout status deployment/prometheus-deployment -n monitoring

### (동적 환경설정)

##### 1. monitoring-namespace 설정하기
    $ cd project2/provision/kubernetes/operator
    $ kubectl apply -f monitoring-namespace.yaml

##### 2. 프로메테우스 오퍼레이터 배포
    $ kubectl apply -f prometheus-operator-rbac.yaml

##### 3. 새로운 계정 설정 후 추가 작업
    $ kubectl apply -f prometheus-operator-deployment.yaml 
    $ kubectl rollout status deployment/prometheus-operator -n monitoring

#### 프로메테우스 서버 배포
##### 1. 올바른 액세스 제어 권한을 가진 인스턴스를 부여
    $ kubectl apply -f prometheus-tbac.yaml
##### 2. 서버 배포
    $ kubectl apply -f prometheus-server.yaml
##### 3. 배포 상태 확인
    $ kubectl rollout status statefulset/prometheus-k8s -n monitoring
##### 4. 새로운 서비스 생성 및 웹 인터페이스 시작
    $ kubectl apply -f prometheus-service.yaml
    $ minikube service prometheus-service -n monitoring

#### 프로메테우스 타깃 추가
##### 1. hey를 통해 사용 가능한 타깃의 수 늘리기
    $ kubectl apply -f hey-deployment.yaml
    $ kubectl rollout status deployment/hey-deployment -n monitoring
##### 2. 서비스 모니터에서 참조하는 레이블에 주의
    $ kubectl apply -f hey-service.yaml
##### 3. 서비스 실행
    $ minikube service hey-service -n default
##### 4. hey 애플리케이션에 대한 서비스 모니터 생성
    $ kubectl apply -f prometheus-servicemonitor.yaml
    $ kubectl apply -f hey-servicemonitor.yaml
##### 5. 서비스 모니터 pod 배포 상태 나타내기
    $ kubectl get servicemonitors --all-namespaces
    

