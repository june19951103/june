#### Podman 이란?

- Red Hat Enterprise Linux 8 / Cent OS 8 부터는 Docker 대신 Podman 이라는 도구를 제공
Docker와 동일하게 단일 노드에서 Pod, 컨테이너 이미지 및 컨테이너르 관리
libpod 라이브러리를 기반


#### podman 설치


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

