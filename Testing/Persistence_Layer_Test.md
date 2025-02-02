JPA를 사용할 때, 예외 조건이 많은 등의 상황에서 쿼리 메서드가 길어지게 되면 파라미터를 잘못 주는 상황이 발생할 수 있다. 또한, 구현하는 기술의 변경(MyBatis, QueryDSL …)이 일어날 때 개발자가 작성한 메서드에서 제대로 된 쿼리가 날라갈 것인가에 대한 보장을 하기 위함이다.  

> Repository Test는 단위 테스트의 성격에 가까운 테스트이다.
> 

물론 Spring 서버를 띄워서 테스트를 진행하지만, Database에 접근하는 로직만 가지고 있기 때문에 기능 단위로 보면 단위 테스트의 성격을 가지고 있다. 

### `@SpringBootTest`

> 테스트를 진행할 때, Spring 서버를 구동시켜 테스트를 진행하게 된다.
> 

### `@DataJpaTest`

> Spring 서버를 띄워 테스트하지만, SpringBootTest 보다 가볍다.
> 
- JPA에 관련된 Bean들만 주입해 서버를 띄워 속도가 빠르다는 장점이 있다.

# List에 대한 체크 순서

1. 리스트의 사이즈 확인
2. 검증하고자 하는 필드 추출
3. 결과 확인

```java
assertThat(products).hasSize(2)
		.extracting("productNumber", "name", "sellingStatus")
		.containsExactlyInAnyOrder(
			tuple("001", "아메리카노", SELLING),
			tuple("002", "카페라떼", HOLD)
		);
```

- `extracting` := 검증하고 싶은 필드를 추출하는 메서드
- `containsExactlyInAnyOrder` := 순서에 상관없이 값이 일치하는지만을 확인하는 메서드
- `tuple` := 이 메서드를 통해 원하는 값들을 확인

# 전체 코드

```java
@DataJpaTest
@ActiveProfiles("test")
class ProductRepositoryTest {

	@Autowired
	private ProductRepository productRepository;

	@DisplayName("원하는 판매 상태를 가진 상품들을 조회한다.")
	@Test
	void findAllBySellingStatusIn() {
		// Given
		Product product1 = Product.builder()
			.productNumber("001")
			.type(HANDMADE)
			.sellingStatus(SELLING)
			.name("아메리카노")
			.price(4000)
			.build();

		Product product2 = Product.builder()
			.productNumber("002")
			.type(HANDMADE)
			.sellingStatus(HOLD)
			.name("카페라떼")
			.price(4500)
			.build();

		Product product3 = Product.builder()
			.productNumber("003")
			.type(HANDMADE)
			.sellingStatus(STOP_SELLING)
			.name("팥빙수")
			.price(7000)
			.build();
		productRepository.saveAll(List.of(product1, product2, product3));

		// When
		List<Product> products = productRepository.findBySellingStatusIn(List.of(SELLING, HOLD));

		// Then
		assertThat(products).hasSize(2)
			.extracting("productNumber", "name", "sellingStatus")
			.containsExactlyInAnyOrder(
				tuple("001", "아메리카노", SELLING),
				tuple("002", "카페라떼", HOLD)
			);
	}
```

- `@ActiveProfiles("test")` := application.yml 파일에서 profile이 test인 부분을 사용해서 실행