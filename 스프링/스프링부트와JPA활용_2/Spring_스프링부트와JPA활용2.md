```java
    public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o"
                        + " join fetch o.member m"
                        + " join fetch o.delivery d"
                        + " join fetch o.orderItems oi"
                        + " join fetch oi.item i", Order.class)
                .getResultList();
    }
```

1. DB의 distinct를 날려준다.
   * 하지만 DB의 distinct는 모든 컬럼이 같아야 생략된다.
2. from 의 객체가 중복된 경우에 생략해준다.



fetch join과 페이징 처리하면 메모리에서 처리함.

즉, 결과가 1만개면 만개를 모두 메모리에 올리고 페이징 처리를 하게 됨.

그리고 또 다른 문제는 일대다인 경우 DB 쿼리가 뻥튀기가 되어있는데 그 상태에서 결국 중복을 걸러주지 못하고 DB 쿼리에서 가져온 결과(위에 나온 사진과 같이 4개)를 기준으로 페이징하게됨.

컬렉션 페치 조인은 1개만 사용할 수 있다. 즉, 일대다 관계가 2개 이상이면 JPA자체가 맞춰줄 수 없을 수 없다(너무 많이 row가 많아지고 JPA가 기준을 잡지 못함) 

