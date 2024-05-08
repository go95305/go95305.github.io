---
layout: single
title:  "Gitlab Container 3rd party 서비스 소유자 및 그룹을 변경하여 실행하기"
category:
  - Git
  - Docker
  - Linux
  - 성장기
---

<br/>

## Gitlab Container 3rd party 서비스 소유자 및 그룹을 변경하여 실행하기
<br>

gitlab omnibus배포판으로 설치할 경우 3rd party 서비스들도 함께 설치되며 손쉽게 사용할 수 있는 장점 존재.

### 테스트 정보

- Linux 환경(RHEL 7.9)에서 gitlab.rb수정을 통한 3rd party 서비스들의 실행 주체를 변경하는 테스트 진행.

- /var/opt/gitlab 하위에 있는 디렉토리들의 owner, group에 대한 정보도 같이 수정됨.

### 테스트 순서
>1. gitlab/gitlab-ce:15.7.8-ce.0 버전의 이미지로 컨테이너 실행.
>2. **useradd -u 2124 gitlab** 으로 새로운 계정 생성 후 docker commit으로 새로운 이미지버전인 15.7.8-ce.0.1 생성
>3. postgresql[uid] = gitlab(기존 gitlab-psql), git[uid] = gitlab(기존 git) 등 **uid** 및 **gid**를 수정.
>4. 위의 수정과정은 gitlab.rb에서 진행. 물론 컨테이너 내부에 있는 gitlab.rb를 어떻게 실행전에 수정하느냐가 문젠데, 기존 초기 컨테이너 실행시 Volume Mount를 통해 host쪽에 접근가능하게 설정.
>5. 예를 들면 /app/gitlab/config/gitlab.rb이 volume mount공간으로 되어있다면, docker run command에 위의 경로를 -v 옵션을 줘서 실행.  
>6. **15.7.8-ce.0.1** 버전으로 새로운 컨테이너 실행
>7. 컨테이너 내부에 들어가서 확인해보면 새로운 uid, gid로 디렉토리 설정이 된것을 확인가능.

