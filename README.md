# Server-DevOps

AWS, Jenkins, Docker에 대한 사용법을 정리하고 서버 운영에 관한 내용을 정리하는 공간
--------------------------------------------------------------------


### AWS 
    
  1. EC2
   
    AWS를 이해하기 위해 EC2는 하나의 컴퓨터라고 생각하면 된다. 가상 환경에서 다른 컴퓨터 서버를 동작시키는데 그것을
    EC2의 인스턴스를 띄운다고 한다. EC2의 인스턴스는 기본적으로 리눅스 프로그램을 따르고 있어 그에 관한 명령어로 동작시켜야 한다.
    EC2를 생성시켰다면, 이를 껐다가 실행할 수 있고 여기서 보안그룹으로 들어가 포트를 설정할 수 있다.
    ex) 0.0.0.0/:0 or //:0를 선택하고 + port(아무 번호)를 설정하게 되면 port에 대한 번호를 가져올 수 있다는 의미를 가진다.
       
  2. 탄력적 IP
    
    탄력적 IP란? IP의 사이즈가 줄어들었다가 커졌다가 실행시키는 프로그램에 따라 가격을 측정할 수 있게 만드는 것이다. 
    이게 왜 필요하냐고 물어보면 이는 EC2와 관련이 있다. EC2의 인스턴스를 사용을 할 때 이를 중단시켰다가 실행시켰다가 할 수 있다.
    근데 중단시켰다가 다시 실행을 시키게 되면 IP주소가 바뀌는 경우가 있다. 이는 하나의 문제가 된다. 
    왜냐하면 기존에 설정되어 있는 IP가 바뀌게 되면 그에 따라 설정된 프로그램의 IP 주소도 바꿔야 하기 때문이다.
    이러한 문제를 해결하기 위해 탄력적 IP를 사용하게 되면 인스턴스를 중단시켰다가 다시 실행하더라도 IP주소가 바뀌지 않아 사용가능하게 된다.
    
  3. EC2 실행시키기
  
    환경 : macOS, example.pem(인스턴스 생성시 다운받을 수 있는 암호화된 키), example.amazon.com DNS가 존재
    실행 코드
    1. example.pem이 있는 폴더로 간다. => cd Documents/key
    2. EC2 실행 => ssh -i "example.pem" ec2-user@example.amazon.com 실행 그리고 yes
    3. EC2가 샐행됨 [example]$ 상태가 된다.
    
    
### Jenkins

  0. Jenkins가 필요한 이유
  
    왜 Jenkins가 필요할까 그 이유는 이를 활용해 배포관리를 원활히 할 수 있기 때문이다. 
    코드가 배포되기 전에 오류를 잡아서 알려주고 잘못된 점이 있다면 우리에게 알려줘 오류를 쉽게 잡을 수 있게 만든다.

  1. AWS EC2에 Jenkins 띄우기
  
    EC2에 Jenkins를 띄우는 것은 배포관리를 할 때에 다른 개발자들과 Jenkins로 소통을 할 때에 중요한 이슈이다.
    그래서 이제는 어떻게 EC2에 Jenkins를 띄울 수 있는지 알아보도록 하겠다.
    환경 : AWS 3번을 먼저 하고 [example]$ 상태로 만들어야함/Linux 환경은 데비안과 CentOS 계열이 있는데 우리는 CentOS임
    추가 : 데이안 계열 명령어 apt or apt-get/CentOS 명령어 yum임 다르다는 것을 알고 사용해야함
    실행 코드
    1. yum을 업데이트 => [example]$ sudo yum update -y
    2. Jenkins 더해줌 => [example]$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
    3. key file import => [example]$ sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
    4. Install Jenkins => [example]$ sudo yum install jenkins -y
    5. Jenkins 실행 => [example]$ sudo service jenkins start
    
    오류
    - 5번을 실행할 때 오류가 나며 실패할 때가 있다 이때 Linux 시스템에 openjdk 가 존재하지 않아 openjdk를 다운받게 되는데 이때 조심해야 함
    - https://stackoverflow.com/questions/39621263/jenkins-fails-when-running-service-start-jenkins/45430023#45430023
    - 위 링크에 나온 것처럼 apt-get을 사용하게 되면 발생하게 되는 문제가 바로 CentOS에서 사용하지 않는 명령어가 apt이기 때문이다. 
    - https://bamdule.tistory.com/57
    - 위 링크처럼 sudo yum install java-1.8.0-openjdk를 사용해 다운받고 설정해줘야 한다.
    - 그리고 여기서는 vi 명령어를 사용했으나 Linux 시스템이므로 sudo cat /etc/profile로 설정해야 한다.
    => 그러면 5번에서 발생하는 오류는 해결하게 된다.
    
    6. Unlock Jenkins 화면 비밀번호를 찾아 복사해서 붙여넣는다 => [example]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    7. 도메일 설정 어차피 설정된 상태로 나오게 된다 => http://<your_server_public_DNS>:8080 
    
  2. Local DockerHub에서 Jenkins 띄우기
  
    - https://hub.docker.com/r/jenkins/jenkins/
    - https://github.com/jenkinsci/docker/blob/master/README.md#upgrading-plugins
    
  3. 가장 최적의 Jenkins
  
    - 위 방법들은 따로따로 하게 되어 있는데 사실 EC2 안에 Docker 그리고 그 안에 Jenkins를 띄우는 것이 가장 최적의 방법이다.
    
  4. git과 연동하기
    
    환경 : AWS, MacOS
    코드실행
    0. AWS 가상 머신에서 git 설치(없다면) => sudo yum install git && which git(경로파악)
    1. Jenkins에서 git 경로설정
    2. https://goddaehee.tistory.com/258
    3. Jenkins에 WebHook 추가하기
       - Webhook이란? Webhook(웹훅)이란, 서버에서 어떠한 작업이 수행 되었을 때 해당 작업이 수행되었음을 HTTP POST 로 알리는 개념
    
    주의사항 
    - Username with Password 할떄 Username은 닉네임임
    - 오류 Couldn't find any revision to build. Verify the repository and branch configuration for this job. : https://huskdoll.tistory.com/484
    

### Docker
    - Docker란 Go 언어로 작성된 리눅스 컨테이너 기반으로 하는 오픈소스 가상화 플랫폼
    - Docker를 통해 여러 개의 EC2에 같은 버전의 가상환경을 만들 때 효과적으로 제어할 수 있고 동시성을 유지하기 편함
    
  1. Docker File 만들기
    
    - Docker File은 Docker Image를 만들기 위해 만든 설정 파일로 여러가지 명령어를 토대로 Docker File을 작성하면 설정된 내용대로 Docker Image를 만들 수 있음
    
    ex) Docker File 작성 예
    
    $ vim Dockerfile
    
    FROM ubuntu:14.04
    
    # app 디렉토리 생성

    
