
---
layout: single
title:  "Entity 수정(Setter를 사용하지 않는 방법)"
---

boardRepository.save("엔티티")를 통해 DB에 존재하는 컬럼값을 수정할 수 있습니다.
아무래도 데이터 삭제보다는 삭제 되었다 라는 표시(?)를 남겨두고 게시글을 가져올땐 "삭제되었다" 라는 표시가 false인것들만 불러오고 싶었다.

아래 코드를 보면 findById를 통해 특정 레코드를 가져온다. → 다음으로 해당 Entity를 Dto로 바꾸고 isDeleted를 true로 설정해준다.(삭제 여부) → Dto를 다시 Entity로 바꾸고 save를 통해 영속상태인 Entity를 update해준다.

<p align="center">
    <img src="/images/screen_20220225.png" width="50%" class="image__border">
</p>

하지만 좀 더 간단하게 할 수 없을까 라는 고민을 하게 되었다. 물론 Setter를 통해 직접적으로 Entity의 값을 수정할 수는 있지만 그런 방식은 추천하지 않는다. 그 이유는 아래서 따로 설명하겠다.

결국 1.더욱 간단하게 작성 2. Entity자체에 Setter를 사용하지않기
위의 조건을 만족하기 위해 메소드를 만들어서 Entity내에서 처리하도록 수정하였다. 코드도 짧아지고 좀더 안전한(?) 코드가 되었다.

<p align="center">
    <img src="/images/screen_20220225_2.png" width="50%" class="image__border">
</p>

<br/>



**Entity에 Setter를 두게 되면 일관성을 유지하기 힘들어 위험하기 때문이다.**

한마디로 간단한 코드에서는 수정 혹은 삭제를 위해서 setter를 사용하구나! 라는 것을 알 수 있지만 코드가 복잡해지고 여기저기서 entity.setColumn()을 쓰게되면 도대체 어디서 어떤 이유로 setter를 쓰는지 알 수 없기때문에 차라리 메소드를 만들어서 한눈에 어떤 용도인지 정의하는 것이 좋다.

예를 들어 isDeleted 컬럼의 수정을 위해서는,

~~~java
public BoardEntity setDelete(){
        this.isDeleted = true;
        return this;
    }
~~~

와 같이 해주는 것이 좋다.(setDelete가 좋은 메소드명인지는 모르겠지만 삭제과정에 필요하다는 것은 한눈에 알 수 있다.!)