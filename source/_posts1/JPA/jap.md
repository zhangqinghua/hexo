---
title : jpa

categories:
- JAP

date: 2018-05-29
---

```java
@Modifying
@Transactional
@Query(value = "update cl_cook_book_step set is_deleted = 'y' where cook_book_id = ?1", nativeQuery = true)
void deleteByCookBookId(Long cookBookId);
```