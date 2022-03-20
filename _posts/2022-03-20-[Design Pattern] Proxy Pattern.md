---
layout: single
title:  "[Design Pattern] Proxy Pattern"
---

### Proxy Pattern이란?
Proxy객체란 원래 객체(real class)를 대신 구현해준다고 생각하면된다. 직접적으로 원래 객체를 구현하기 보다는 Proxy class 내부에 원래 객체(class)를 선언하면서 Proxy Pattern을 구현할 수 있다.

그렇다면 왜 대신 구현해줄까?

그 이유는 아마도 프록시 객체를 통해서 실질적인 객체의 Service를 구현하므로 보안적인 부분에 장점이 있다고 본다. 네트워크 분야의 시점에서 프록시 서버의 구축의
이유라고 봐도 될거같다.

클라이언트에서 서버로 요청을 보낼때 중간에 Proxy서버를 두게 되면 서버입장에서는 어느 클라이언트에서 요청이 들어온지 비밀(?) 로 할 수있습니다.
 
Proxy 패턴을 간단한 예제로 구현해 보았다.


- review
  - 댓글을 다는 기능이 담긴 Interface
- pos
  - review Interface를 상속받아서 실질적인 댓글작성 기능을 수행하는 원래 객체(real class)
- proxy
  - pos 대신 댓글 작성을 해주는 proxy class

<p align="center">
    <img src="/images/proxy/review_interface.png" width="50%" class="image__border">
</p>
<br/>
<p align="center">
    <img src="/images/proxy/pos.png" width="50%" class="image__border">
</p>
<br/>
<p align="center">
    <img src="/images/proxy/proxy.png" width="50%" class="image__border">
</p>
<br/>
<p align="center">
    <img src="/images/proxy/main.png" width="50%" class="image__border">
</p>

main에서 pos객체를 직접 호출하는 것이아니라 proxy를 통해 '대신' 댓글작성 기능을 수행하는 것이다.
proxy 클래스를 보면 댓글작성기능이 있는 Review Interface를 상속받은 Pos가 구현되어있는것을 확인할 수 있다.






<br/>
<br/>
<br/>
<br/>
