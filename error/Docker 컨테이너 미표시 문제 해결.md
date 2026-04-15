[Docker] 터미널에선 실행되는데 Docker Desktop UI에 안 뜰 때 해결법
Docker Compose로 컨테이너를 실행했을 때, 터미널(CLI) 결과는 성공(Started)인데 Docker Desktop GUI 화면에는 아무것도 나타나지 않는 경우가 있습니다. 이 현상의 원인과 해결 방법을 정리합니다.

1. 원인: Docker Context의 불일치
   도커에는 **Context(컨텍스트)**라는 개념이 있습니다. 쉽게 말해 "명령어를 전달할 도커 엔진의 대상"입니다.

default: WSL2 리눅스 배포판 내부에 별도로 설치된 도커 엔진을 바라봅니다. (Docker Desktop UI와 연동 안 됨)

desktop-linux: Windows의 Docker Desktop 엔진을 바라봅니다. (UI와 연동됨)

터미널에서 명령어를 실행할 때 default 컨텍스트를 사용하고 있다면, 컨테이너는 리눅스 깊숙한 곳에서 잘 돌아가고 있지만 Docker Desktop 앱은 이를 감지하지 못합니다.

2. 해결 방법
   STEP 1. Docker Desktop 설정 확인 (WSL Integration)
   가장 먼저 Docker Desktop 앱이 현재 사용 중인 터미널(Ubuntu 등)과 연결되어 있는지 확인해야 합니다.

Docker Desktop Settings(톱니바퀴 아이콘) 클릭

Resources > WSL Integration 메뉴로 이동

Enable integration with my default WSL distro 체크

하단 리스트에서 현재 사용 중인 배포판(예: Ubuntu)의 스위치를 ON으로 변경

Apply & Restart 클릭

STEP 2. 터미널에서 Context 변경
설정을 마쳤다면 터미널에서 명령어가 전달될 대상을 Docker Desktop으로 바꿔줘야 합니다.

Bash
# 1. 현재 컨텍스트 리스트 확인
docker context ls

# 2. desktop-linux로 컨텍스트 전환
docker context use desktop-linux
만약 context not found 에러가 발생한다면?
Docker Desktop의 WSL Integration 설정이 제대로 적용되지 않은 상태입니다. 앱을 재실행하거나 설정을 다시 한번 확인해 보세요.

STEP 3. 컨테이너 재실행
기존 default 컨텍스트에서 실행했던 컨테이너들을 정리하고, 새로운 컨텍스트에서 다시 실행합니다.

Bash
# 기존에 잘못 띄워진 컨테이너 종료 (필요 시 context 일시 복구 후 종료)
docker context use default
docker-compose down

# 다시 desktop-linux로 설정 후 실행
docker context use desktop-linux
make service-run  # 또는 docker-compose up -d
3. 결과 확인
   이제 Docker Desktop의 Containers 탭을 확인해 보세요. 터미널에서 보았던 zookeeper, redis-db, postgresql-db, kafka 컨테이너들이 프로젝트 이름으로 묶여 정상적으로 표시되는 것을 확인할 수 있습니다.

요약
WSL2 환경에서 개발한다면 터미널의 도커 엔진과 Docker Desktop 앱의 엔진이 서로 다를 수 있음을 인지해야 합니다. docker context ls 명령어를 통해 내가 지금 어디에 명령을 내리고 있는지 확인하는 습관을 들이면 좋습니다!