# Prometheus-practice
 모니터링 프로메테우스 연습
#### * Progress in macOS Monterey 12.3.1
#### * Hardware is MacBook Pro (15-inch, 2016)

 # 필요한 소프트웨어
 ## 1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
: 오픈소스 하이퍼바이저. 가상 시스템을 원하는 이미지로 시작할 수 있음. 또한 가상 네트워크를 만들고 호스트의 파일 시스템 경로를 게스트에 마운트 가능하다.
 ## 2. [Vagrant](https://www.vagrantup.com/downloads)
: 가상 머신을 쉽게 만들고 관리해주는 툴이다.
 ## 3. [Minikube](https://minikube.sigs.k8s.io/docs/start/)
: 로컬 환경에서 쿠버네티스 환경을 구성하고 테스트할 수 있는 도구이다.
 ## 4. [kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-macos/)
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
    $ minikube start \ --cpus=2 \ --memory=2048 \ --kubernetes-version=="v1.23.3" \ --vm-driver=virtualbox
##### 3. 쿠버네티스 대시보드 진입
    $ minikube dashboard
##### * 주의사항
<div align="center">
<img src="[https://user-images.githubusercontent.com/54494793/170031469-b43a1d1a-3486-4df5-a4ad-d8258631a3a5.jpg](https://user-images.githubusercontent.com/54494793/170822639-e9013849-9e5b-4a39-a38d-b31c20a7993c.png)" width="100%" height="100%" style="border-radius:5%;">
</div>
#### Exiting due to HOST_BROWSER: failed to open browser: exit status 1 
#### -> 맥의 기본 브라우저를 chrom에서 safari로 바꾸면 해결됨.

##### * 배포 상태 확인하기
    $ kubectl rollout status deployment/prometheus-deployment -n monitoring

##### * 프로메테우스의 인스턴스의 로그 확인하기
    $ kubectl logs --tail=20 -n monitoring -l app=prometheus
