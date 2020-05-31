#### Podman 이란?

- Red Hat Enterprise Linux 8 / Cent OS 8 부터는 Docker 대신 Podman 이라는 도구를 제공
Docker와 동일하게 단일 노드에서 Pod, 컨테이너 이미지 및 컨테이너르 관리
libpod 라이브러리를 기반

#### podman & Docker

- 공통점
리눅스에서 컨테이너를 사용할때 사용하는 오픈소스 tool.
명령어도 똩같기 때문에 이미지의 pull, 컨테이너 실행 명령어들도 동일하게 사용가능.

- 차이점
가장 차이는 데몬의 존재여부이다.
기존 OS위에 가상화 하지않고 서비스를 올릴 수 있다는 점.
하지만 docker의 경우 데몬이 꺼지면 모든 컨테이너 역시 종료된다.
podman은 그 점을 보완해 개별적 on/off 가 가능하다.

#### podman 설치 및 간단한 


$ yum -y install podman

- podman은 systemd 설정 할 필요가 없다.

$ podman run -ti -d web httpd

- 위 구문과 같이 컨테이너 생성이 가능.

$ podman ps -a 

- 위 구문으로 컨테이너 상태 확인가능.

$ podman stop web 

- stop 이라는 옵션을 통해 컨테이너 중지 가능하며 ps 구문을 통해 상태 확인이 가능.

$ podman start web 

- 중지된 컨테이너는 start 옵션을 통해 컨테이너 시작이 가능.

$ podman images

- 현재 Host에 Pull 된 컨테이너 이미지를 확인 가능.

$ podman pull centos

- RedHat Registry 혹은 Docker Hub의 컨테이너 이미지를 Pull 할 수 있다.

