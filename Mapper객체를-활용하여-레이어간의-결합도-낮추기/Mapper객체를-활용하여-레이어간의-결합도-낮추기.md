## 각 계층간의 결합도를 낮출수는 없을까 ? 

프로젝트 설계에서 각 계층이 서로에게 영향을 주지 않도록 분리하는 것은 핵심적인 고민 사항 중 하나입니다. 서비스의 규모가 커지고, DTO가 늘어날수록 각 계층간의 데이터 바인딩 작업은 복잡해집니다. 만일 하나의 DTO 또는 엔티티가 많은 바인딩 책임을 지게 된다면, 해당 객체는 과도한 책임을 떠안게 됩니다.

![](https://velog.velcdn.com/images/leon/post/3a91d6fb-c015-44cf-8291-d5217337c017/image.png)

![](https://velog.velcdn.com/images/leon/post/102d876c-d6df-4379-9be7-3c0415c9113f/image.png)

> 이러한 구조는 다음과 같은 문제점을 내포하고 있습니다.
1. DTO가 추가될 때마다 메서드를 계속 추가해야 합니다.
2. 해당 엔티티가 너무 많은 DTO를 참조하고 있습니다.
3. 해당 엔티티에 과도한 책임이 부여됩니다.


## Mapper

이런 고민에 대한 해결책으로, 각 계층의 분리를 확실히 하기 위해 Mapper 객체를 도입하였습니다. Mapper는 계층 간 이동 시 데이터를 해당 계층의 전용 객체(DTO) 또는 엔티티로 변환하는 역할을 수행합니다. 이 방식을 통해 각 데이터는 서로에게 영향을 주지 않고, 해당 데이터에 대한 비즈니스 로직에만 집중할 수 있게 되었습니다. 또한, 데이터의 불변성도 확보할 수 있었습니다.


   ![](https://velog.velcdn.com/images/leon/post/b56226b9-ab3b-45fe-ad0c-69bfaef1142f/image.png)

![](https://velog.velcdn.com/images/leon/post/5a9187c9-9c37-453c-af0a-783e6f5c86b4/image.png)

![](https://velog.velcdn.com/images/leon/post/a7d53b54-f10e-4fa6-afcd-07c6bad7dc77/image.png)

Mapper의 사용은 데이터 생성 시점에서의 유효성 검사(validation) 수행도 가능하게 합니다. 이를 통해, 비즈니스 로직 수행 전에 잘못된 데이터가 발생하는 것을 사전에 방지할 수 있습니다.

![](https://velog.velcdn.com/images/leon/post/176fab30-71ec-4d4a-8bdd-70f67b2f367a/image.png)


![](https://velog.velcdn.com/images/leon/post/930bdd0b-2a0a-43b8-84c2-243086ae707f/image.png)
