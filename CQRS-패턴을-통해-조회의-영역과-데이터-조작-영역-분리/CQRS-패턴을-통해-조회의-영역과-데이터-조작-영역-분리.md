## CQRS 패턴 사용하게 된 이유

기존에는 조회와 데이터 조작을 다루는 작업을 하나로 했었습니다. 그로 인해 서비스가 커질수록 CUD 작업과 Read 작업이 섞여 복잡도가 높아지고, 유지보수가 어려워 졌던 경험이 있습니다. 따라서 기능에 더 가까이 종속되고, 그에 따라서 나타나는 데이터가 달라질 수 있는 읽기를 담당하는 레포지토리와 엔티티를 직접적으로 참조해서 조작하는 삽입, 업데이트, 삭제를 담당하는 레포지토리를 분리하여, 복잡도와 책임 분리를 개선하고자 했습니다. (CQRS 패턴 레벨 1단계 적용)

![](https://velog.velcdn.com/images/leon/post/52568014-e7b8-464d-ad15-d07de0e8cffe/image.png)

### 서비스단까지 Read작업과 CUD 작업을 나눴을 경우

만약에 서비스까지 Read와 CUD작업을 나누는 구조를 했다면, 필히 CUD작업을 하는 서비스단에서 Read를 해야하는 상황이 옵니다. 그럴 경우에는 단순 Read만 해주는 Util 클래스를 만들어 해결할 수 있습니다.

![](https://velog.velcdn.com/images/leon/post/488225c6-7f5c-4294-b27f-2226034e1f0c/image.png)
![](https://velog.velcdn.com/images/leon/post/b4b50d3e-a0a4-4dc0-b63f-ae84f968b07d/image.png)


### 레포지토리 영역만 Read, CUD 나눴을 경우

레포지토리 단만 Read작업과, CUD 작업을 나눳을 경우엔 다중상속을 통해 서비스레이어에서는 하나의 레포지토리를 바라보게 함으로써 구현할 수 있습니다.

 ![](https://velog.velcdn.com/images/leon/post/cd70e5f6-f554-4a5e-98a6-afa5749cb361/image.png)
