---
layout: single
title:  "[또또가 프로젝트] 검색 기능 리팩터링 "
published : false
categories: ttottoga_project
toc: true
---

## 검색 기능

- 이전에는 아주 단순히 JPA 만을 이용한 검색 기능을 개발

```java
List<StoreResultResponseDto> stores =
                storeRepository.findByTitleContainingOrNameContaining(keyword, keyword).stream()
                        .sorted(comparator())
                        .map(store -> new StoreResultResponseDto(store.getId(), store.getTitle(),
                                store.getImage(), store.getServiceInfo(), store.getReviewCount(),
                                heartStoreRepository.findByMemberAndStore(memberEntity, store).isPresent())).toList();
```

- 쿼리문

```sql
select
        se1_0.id,
        se1_0.address,
        se1_0.created_at,
        se1_0.hot_yn,
        se1_0.image,
        se1_0.menu_id,
        se1_0.modify_at,
        se1_0.name,
        se1_0.place_info,
        se1_0.region_id,
        se1_0.review_count,
        se1_0.review_span,
        se1_0.sale_info,
        se1_0.service_info,
        se1_0.spon_info,
        se1_0.sub_title,
        se1_0.title,
        se1_0.use_info 
from
        store se1_0 
where
        se1_0.title like ? escape '\' 
        or se1_0.name like ? escape '\'

```

- 쿼리문 보면 검색어 2개를 받아서, title 기준으로 검색하고 name 검색
- 즉, '키워드' != '키 워 드' 다르게 인식한다는 것. 공백 포함과 아닌 것은 차이가 나는 것

> 내가 개선하고 싶은 부분
>
> 띄어쓰기 상관 없이 같은 결과를 보여주고 싶음
>
> '검색키워드' == '검색 키워드' 같은 검색 결과가 나오도록

### try 1 받은 키워드를 여러 개로 쪼개서 쿼리문 만들기

