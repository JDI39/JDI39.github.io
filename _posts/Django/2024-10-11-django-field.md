---
title: 3. Django의 필드
author: jdi39
date: 2024-10-11
categories: [Django]
tags: [Django]
toc: true
description: Django의 필드
---

본 포스팅은 '[백엔드 개발을 위한 핸즈온 장고](https://product.kyobobook.co.kr/detail/S000202404727)'을 읽고 공부한 내용을 정리한 것입니다. 

## 장고의 필드
필드는 DB 테이블의 컬럼을 표현하는 추상 클래스로, 장고에서는 DB 테이블을 생성(**db_type()**)하거나 파이썬 타입과 DB를 매핑(**get_prep_value(), from_db_valud()**)하는 데 사용함

모델 내에서 필드는 클래스 attribute로 인스턴스화 되거나 테이블의 특정 컬럼을 표현

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

### Primary Key 관련 필드 
장고 모델로 생성되는 테이블은 primary key를 반드시 가지도록 설계; primary key 미지정 시, 장고가 primary_key=True 옵션을 가지는 필드를 생성 

1. AutoField
- 파이썬 자료형: int 
- 데이터베이스 자료형: int 

  available ID에 따라 자동으로 생성되는 IntegerField, primary_key=True인 필드가 존재하지 않으면 자동으로 AutoField를 생성

```python
# 두 모델은 동일한 결과를 보여줌
class DjangoModel1(Model):
  id = models.AutoField(
    auto_acreated=True, primary_key=True, serialize=False, verbose_name='ID',
  )

class DjangoModel2(Model):
  # id = models.AutoField(
  #   auto_acreated=True, primary_key=True, serialize=False, verbose_name='ID',
  # )
```

2. BigAutoField
- 파이썬 자료형: int
- 데이터베이스 자료형: int 

  AutoField에 비해 더 많은 자리수를 보장 (1 ~ 9,223,372,036,854,775,807)

```python

```

3.   



#### 참고문헌
https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.Field
https://docs.djangoproject.com/en/5.1/ref/models/fields/#autofield
https://docs.djangoproject.com/en/5.1/topics/db/models/
