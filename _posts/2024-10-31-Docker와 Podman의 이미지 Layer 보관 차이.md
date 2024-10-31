---
layout: single
title:  "Docker와 Podman의 이미지 Layer 보관 차이"
category:
  - Docker
  - In-Job
  - Podman
---

<br/>

## Docker의 Image Layer 보관

Docker와 Podman의 가장 큰 차이는 'Daemonless 구조'라고 볼수있다.
Docker는 기본적으로 Docker Daemon에 의해서 Container, Image, Registry 관리(접근)이 가능하다.
반면, Podman은 유저 기반으로 Daemon없이도 접근이 가능하다는 차이점이 있다.

<p align="center">
    <img src="/images/docker_architecture.png" width="100%" class="image__border">
</p>

<p align="center">
    <img src="/images/podman_architecture.png" width="100%" class="image__border">
</p>



<br>

### Container Image 삭제 시 차이 
<br>

Docker에서 이미지 삭제시(docker rmi) 여러 이미지가 공유하지 않는 Layer만 삭제한다. 하지만 Podman의 경우, 이미지가 user의 home directory (~/.local/share/containers/storage)에 저장되므로, 이미지 삭제 시 해당 이미지의 Layer를 공유하는 다른 이미지가 타격을 입을 수도 있다.
정확히는 **다른 User의 Layer저장위치의 접근 권한 문제라고 볼 수도 있다.**


