# Prometheus-practice
 모니터링 프로메테우스 연습

 # 필요한 소프트웨어
 1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
 </br>
    : 오픈소스 하이퍼바이저. 가상 시스템을 원하는 이미지로 시작할 수 있음. 또한 가상 네트워크를 만들고 호스트의 파일 시스템 경로를 게스트에 마운트 가능하다.
 2. [Vagrant](https://www.vagrantup.com/downloads)
 </br>
    : 가상 머신을 쉽게 만들고 관리해주는 툴이다.
 3. [Minikube](https://minikube.sigs.k8s.io/docs/start/)
 </br>
    : 로컬 환경에서 쿠버네티스 환경을 구성하고 테스트할 수 있는 도구이다.
 4. [kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-macos/)
 </br>
    : 쿠버네티스 클러스터를 제어하기 위한 커맨드 라인 도구이다.




<br/>

# 1. 환경설정 연습하기
#### 테스트 환경 구축
    $ vagrant up
#### vagrant running 확인
    $ vagrant status

</br>

#### 프로메테우스, 그라파나, 알림 메니저의 HTTP 엔드포인트
    192.168.56.10:9090/targets
    192.168.56.11:3000
    192.168.56.12:9093

#### 테스트 환경 파괴
    $ vagrant destroy -f