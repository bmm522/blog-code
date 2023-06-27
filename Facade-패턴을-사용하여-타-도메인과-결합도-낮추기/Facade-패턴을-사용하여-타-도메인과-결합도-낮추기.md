하나의 도메인 서비스에 타 도메인과 연관관계가 있으면 협력하여 비즈니스 로직을 구성해야 할때가 있습니다. 이 상황에서 하나의 도메인 서비스는 그 해당 도메인에 대한 비즈니스 로직만 처리하고, 여러 협력해야하는 비즈니스 로직의 책임은 Facade 패턴의 객체에게 위임하여 책임을 분리할 수 있습니다.

![](https://velog.velcdn.com/images/leon/post/255e651d-2b86-4efe-ab89-c29dde67652e/image.png)

보시다피시 퀴즈 서비스에서는 서로 연관관계가 있는 카테고리 repository, 퀴즈선택지 repository가 의존성주입이 되어있습니다.
하지만 이는 퀴즈서비스에서 타 도메인 객체까지 직접적으로 다루게 되면 추후에 서비스의 사이즈가 커질 수록 의존관계의 복잡도가 높아질 수 있습니다.

![](https://velog.velcdn.com/images/leon/post/4b752253-b614-421f-80e4-2fdaa108328c/image.png)

여러 도메인에 관련된 비즈니스 로직을 처리를 총괄적으로 처리 할 Facade 객체를 생성하여 해당 객체에게 위임합니다.

![](https://velog.velcdn.com/images/leon/post/f6fddd1c-197c-4b6c-8ab7-603b901b466c/image.png)


![](https://velog.velcdn.com/images/leon/post/7569130f-7d91-41d4-a1d4-2a2ee03d67a9/image.png)


그로인해 퀴즈 서비스에서는 해당 도메인에 관련된 비즈니스 로직만 처리가 가능하게 되었습니다.
해당 이슈 https://github.com/bmm522/quiz-studio/issues/23