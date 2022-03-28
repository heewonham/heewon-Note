

### Escape

HTML 문서는 `<` ,` >` 같은 특수 문자를 기반으로 정의된다. 따라서 뷰 템플릿으로 HTML 화면을 생성할 때는 출력하는 데이터에 이러한 특수 문자가 있는 것을 주의해서 사용해야 한다.

는 < 를 HTML 테그의 시작으로 인식한다. 따라서 < 를 테그의 시작이 아니라 문자로 표현할 수 있는 방법이 필요한데, 이것을 HTML 엔티티

고 이렇게 HTML에서 사용하는 특수 문자를
HTML 엔티티로 변경하는 것을 이스케이프(escape)라 한다. 그리고 타임리프가 제공하는 th:text ,
[[...]] 는 기본적으로 이스케이스(escape)를 제공

`<`   -> `&lt`

`>` : `&gt`



### Unescape

`th:text` `th:utext`
`[[...]]`  `[(...)]`



escape가 기본인 이유: 게시판을 만든다고 했을 때, 사용자는 다양한 글을 올린다. 만약 excape 되지 않으면 사용자 작성 글이 깨질 가능성이 있기 때문이다.

실제 서비스를 개발하다 보면 escape를 사용하지 않아서 HTML이 정상 렌더링 되지 않는 수 많은 문제가
발생한다. escape를 기본으로 하고, 꼭 필요한 때만 unescape를 사용하자



### 변수 표현식 (springEL)

- Object

- ${user.username} = userA
- ${user['username']} = userA
- ${user.getUsername()} = userA

- List

- ${users[0].username} = userA
- ${users[0]['username']} = userA
- ${users[0].getUsername()} = userA

- Map

- ${userMap['userA'].username} = userA
- ${userMap['userA']['username']} = userA
- ${userMap['userA'].getUsername()} = userA



변수 선언

```html
<h1>지역 변수 - (th:with)</h1>
<div th:with="first=${users[0]}">
 <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
</div>
```



### 기본객체





```javascript
<!-- username = userA:라고나오기때문에오류 -->
<!-- var username = "[[${user.username}]]" 으로 사용-->
var username = [[${user.username}]];
var age = [[${user.age}]];
```





타입 에러가 발생할 경우 스프링이 자동으로 fielderror를 생성해서 만들어놓기 때문에 

검증오류시사용자 값을 불러와 보여줄 수 있다.

하지만 이런 에러가 아닌경우에는

```java
new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다.")
```

설정해둬야 한다.



파일은 보통 데이터베이스에 저장하지 않는다. 스토리지에 저장하고

데이터베이스에는 파일이 저장된 경로만 저장한다.  경로의 fullpath보다 상대 경로만 저장한다. 



헤더없이 보내면 다운이 안된다.

```java
// 추가
String contentDisposition = "attachment; filename\""+uploadFileName+"\"";


return ResponseEntity.ok()
        .body(resource);
```



파일명이 한글, 특수문자 깨지지 않기 위해서 추가해야한다.

```java
        String encodeUploadFileName = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8);
        String contentDisposition = "attachment; filename\""+encodeUploadFileName+"\"";

```



엔티티를 직접 노출할 경우 

-> 지연로딩의 경우에는 실제 엔티티대신 프록시가 존재하고 jackson 라이브러리는 기본적으로 이 프록시 객체의 json을 알지 못하기 때문에 예외가 발생한다.

그러므로 Hibernate5Module을 설치해야한다.

Hibernate5Module은 기본적으로 초기화된 프록시 객체만 노출한다. 초기화하지 않은 객체는 노출하지 않는데 설정을 바꾸면 모든 객체를 강제로 가져올 수 있다.

```java
	@Bean
	Hibernate5Module hibernate5Module(){
		Hibernate5Module hibernate5Module = new Hibernate5Module();
        Hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);
		return hibernate5Module;
	}
```



옵션을 설정하지 않고 원하는 값을 가져오는 방법은

반환하기 전에 강제로 초기화를 시켜는 방법이다.

```java
    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1(){
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for (Order order : all) {
            order.getMember().getName(); // 강제 초기화
            order.getDelivery().getAddress(); // 강제 초기화
        }
        return all;
    }
```



V3는 재사용성이 높지만

V4는 재사용성은 낮다. 하지만 성능 최적화로는 더 좋다. 그리고 화면이 바뀌면 바꿔야하는 단점을 가지고 있다. 

애플리케이션 전체 관점에서 보았을 때 - 성능이 엄청 차이 나지 않는다.
