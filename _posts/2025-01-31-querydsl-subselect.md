---
layout: post
title: QueryDSL에서 FROM 절에 서브쿼리를 넣어보자 (feat. @Subselect)
categories: wiki
subtitle: 
tags: [Java, SpringBoot, JPA, QueryDSL, Subselect]
---
현재 진행하는 프로젝트에서 쿠폰 관련 복잡한 쿼리를 작성하던 중이었다. 
SQL로 먼저 작업을 해보는데 서브쿼리를 from 절에 넣어야 데이터가 제대로 나오는데, QueryDSL에서는 from 절에 서브쿼리를 넣을 수 없다는 문제를 있었다.
그래서 해결 방법을 찾아보다가 `@Subselect`라는 Hibernate 기능을 사용하면 쿼리를 캡슐화해서 View를 만들어 엔티티처럼 사용할 수 있다는 걸 알게 되었다.
그래서 @Subselect를 활용하여 서브쿼리를 효과적으로 사용하는 방법을 공유하려고 한다.

<img width="743" alt="Image" src="https://github.com/user-attachments/assets/8d09e978-f260-4342-905f-fa789ffb653f" />
<br/>
<br/>


### 서브쿼리로 사용할 클래스 선언

```java
@Subselect(
        "select c.id, ROW_NUMBER() OVER (ORDER BY cd.valid_end_date ASC) as rn " +
                "from coupons c " +
                "inner join coupon_duration_settings cd on c.id = cd.coupon_id " +
                "where c.discount_type = 'RATE' " +
                    "and c.discount_value = (" +
                        "select max(c2.discount_value) " +
                        "from coupons c2 " +
                        "where c2.discount_type = 'RATE'" +
                    ")"
)
@Immutable
@Synchronize("coupons")
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class MaxRateCouponSubDto {

    @Id
    @Column(name = "id")
    private Long id;

    @Column(name = "rn")
    private long rn;
}
```

- select문에서 사용한 컬럼명과 필드에 `@Column` 애노테이션으로 설정해준 name이 동일해야 찾을 수 있다.
- `@Subselect`를 사용한 엔티티는 어떤 특정 엔티티에 종속된 것이 아니기 때문에 `@Immutable`을 선언해주어야 하여 불변(**immutable**), 즉 읽기 전용으로 변경 불가함을 선언해야 한다.
    - `@Immutable` : 해당 엔티티가 변경되지 않는 데이터임을 Hibernate에게 알려주는 애노테이션으로, 변경 감지를 방지하여 성능을 최적화할 수 있다.    
<img width="1003" alt="Image" src="https://github.com/user-attachments/assets/f033f851-4c23-4c52-b86e-e4f52c9e5f45" />
- 또한, 동시성 문제를 제어하여 데이터의 일관성을 보장하기 위해 `@Sychronize`을 사용했다.
<br/>
<br/>


### QueryDSL에서 사용

```java
private static final QMaxRateCouponSubDto maxRateCouponSub = QMaxRateCouponSubDto.maxRateCouponSubDto;
private static final QMaxAmountCouponSubDto maxAmountCouponSub = QMaxAmountCouponSubDto.maxAmountCouponSubDto;

public List<HotelAndCouponResponse> findHotelAndCoupons(HotelSearchConditionDto condition) {
 BooleanBuilder whereClause = new BooleanBuilder();
 ... 
 return queryFactory.select(Projections.constructor(HotelAndCouponResponse.class,
                        hotel.id,
                        Projections.constructor(CouponDto.class,
                                rateCouponDuration.isTimeDeal.stringValue().max(),
                                rateCouponDuration.validStartDate.max(),
                                rateCouponDuration.validEndDate.max(),
                                rateCoupon.discountType.max(),
                                rateCoupon.discountValue.max(),
                                rateCoupon.maxDiscountPrice.max()
                        ),
                        Projections.constructor(CouponDto.class,
                                amountCouponDuration.isTimeDeal.stringValue().max(),
                                amountCouponDuration.validStartDate.max(),
                                amountCouponDuration.validEndDate.max(),
                                amountCoupon.discountType.max(),
                                amountCoupon.discountValue.max(),
                                amountCoupon.maxDiscountPrice.max()
                        )
                ))
                .from(hotel)
                .leftJoin(couponUsageConditionObject).on(couponUsageConditionObject.hotel.id.eq(hotel.id))
                .leftJoin(rateCoupon).on(rateCoupon.id.eq(couponUsageConditionObject.coupon.id)
                        .and(rateCoupon.id.eq(
                                JPAExpressions.select(maxRateCouponSub.id)
                                        .from(maxRateCouponSub)
                                        .where(maxRateCouponSub.rn.eq(1L))
                        )))
                .leftJoin(rateCouponDuration).on(rateCouponDuration.coupon.id.eq(rateCoupon.id))
                .leftJoin(amountCoupon).on(amountCoupon.id.eq(couponUsageConditionObject.coupon.id)
                        .and(amountCoupon.id.eq(
                                JPAExpressions.select(maxAmountCouponSub.id)
                                        .from(maxAmountCouponSub)
                                        .where(maxAmountCouponSub.rn.eq(1L))
                        )))
                .leftJoin(amountCouponDuration).on(amountCouponDuration.coupon.id.eq(amountCoupon.id))
                .where(whereClause)
                .groupBy(hotel.id)
                .fetch();
}
```

- 위에서 생성해준 클래스를 Q클래스로 선언하고, from 절에 넣어 사용하면 `@Subselect`에 작성한 코드가 그대로 서브쿼리로 들어간다.

**SQL 예시**

```sql
SELECT h.id,
       MAX(rcd.is_time_deal),
       MAX(rcd.valid_start_date),
       MAX(rcd.valid_end_date),
       MAX(rc.discount_type),
       MAX(rc.discount_value),
       MAX(rc.max_discount_price),
       MAX(acd.is_time_deal),
       MAX(acd.valid_start_date),
       MAX(acd.valid_end_date),
       MAX(ac.discount_type),
       MAX(ac.discount_value),
       MAX(ac.max_discount_price)
FROM hotels h
         LEFT JOIN coupon_usage_conditions cuc
                   ON cuc.hotel_id = h.id
         LEFT JOIN rate_coupons rc
                   ON rc.id = cuc.coupon_id
                       AND rc.id = (
                           SELECT mrt.id
                           FROM (
                                    select c.id, ROW_NUMBER() OVER (ORDER BY cd.valid_end_date ASC) as rn
                                    from coupons c
                                             inner join coupon_duration_settings cd on c.id = cd.coupon_id
                                    where c.discount_type = 'RATE'
                                      and c.discount_value = (
                                        select max(c2.discount_value)
                                        from coupons c2
                                        where c2.discount_type = 'RATE'
                                    )
                                ) mrt
                           WHERE mrt.rn = 1
                       )
         LEFT JOIN rate_coupon_durations rcd
                   ON rcd.coupon_id = rc.id
         LEFT JOIN amount_coupons ac
                   ON ac.id = cuc.coupon_id
                       AND ac.id = (
                           SELECT mac.id
                           FROM (
                                    select c3.id, ROW_NUMBER() OVER (ORDER BY cd2.valid_end_date ASC) as rn
                                    from coupons c3
                                             inner join coupon_duration_settings cd2 on c3.id = cd2.coupon_id
                                    where c3.discount_type = 'AMOUNT'
                                      and c3.discount_value = (
                                        select max(c4.discount_value)
                                        from coupons c4
                                        where c4.discount_type = 'AMOUNT'
                                    )
                                ) mac
                           WHERE mac.rn = 1
                       )
         LEFT JOIN amount_coupon_durations acd
                   ON acd.coupon_id = ac.id
WHERE h.has_deleted = false
GROUP BY h.id;

```
<br/>
<br/>


### 장점
- 중복으로 여러 곳에서 사용할 경우 하나의 클래스로 두는 것이 가독성이나 유지보수면에서 좋다.
- **읽기 전용(`immutable`)**이기 때문에 불필요하게 엔티티를 저장 및 수정하지 않고 데이터의 일관성을 유지할 수 있다.
- 기존 테이블(여기선 coupons)를 변경하지 않고 새로운 엔티티를 정의하여 사용할 수 있다.
- `@Subselect`는 서브쿼리를 가상의 엔티티로 매핑하는 방식이므로, ORM이 이를 단순한 조회로 처리하기 때문에 복잡한 조인 쿼리보다 성능이 더 나을 수 있다.
<br/>

### 단점
- `@Subselect` 문 안의 쿼리에 동적으로 쿼리를 추가할 수 없다. (제일 아쉬운 부분)
- 쿼리가 복잡한 경우 **성능 저하**를 유발할 수 있다.
- View는 실제 데이터가 존재하지 않기 때문에 **조회만 가능**하다.
<br/>
<br/>


### insight
이번 작업을 통해 QueryDSL에서 FROM 절의 서브쿼리 문제를 해결하는 다양한 방법을 고민할 수 있었다.
처음에는 단순히 JPAExpressions로 해결하려 했지만, 쿠폰처럼 여러 곳에서 중복으로 사용할 경우 성능과 유지보수를 고려할 때 이렇게 @Subselect를 붙인 엔티티를 사용하는 게 더 적절하다는 결론을 냈다.
다행히 쿼리도 기존보다 느려지지 않아서 좋았지만, 이런 상황에서 동적 쿼리가 필요하다면 어떻게 해야 하는지 알아볼 필요가 있다.
<br/>
<br/>
<br/>


**참고**
- [@Subselect Annotation in Hibernate](https://www.baeldung.com/hibernate-subselect)
- [JPA 에서 From 절에 서브쿼리 쓰기(@Subselect 통해서 view 생성)](https://juu-code.tistory.com/49)