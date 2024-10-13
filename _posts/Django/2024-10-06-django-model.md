---
title: 2. 장고의 모델
author: jdi39
date: 2024-10-05
categories: [Blogging, Django]
tags: [Django]
toc: true
description: 장고 모델
---

본 포스팅은 '[백엔드 개발을 위한 핸즈온 장고](https://product.kyobobook.co.kr/detail/S000202404727)'을 읽고 공부한 내용을 정리한 것입니다. 

## ER 모델링

백엔드 개발 시 제일 먼저 진행하게 되는 작업으로, DB 관점에서는 테이블을 정의하고 테이블 관의 관계를 매핑하는 것입니다.

장고 관점에서는 이 테이블 간의 관계를 어떻게 모델로 가져올 것인지를 고민하는 작업입니다.

예를 들어, "서울특별시 서대문구 대현동" 형태의 사용자의 주소(address) 데이터를 처리하는 상황을 가정해보겠습니다. 이 주소를 하나의 문자열로 생각할 수도 있고, '시', '구', '동'이 조합된 형태로 생각할 수도 있습니다. 어떤 방식이 더 적절할지는 고객의 요구 사항이나 기획서를 고려하여 신중하게 결정해야 합니다. 

### 장고로 ER 모델링 표현하기 
장고에서는 모델과 필드를 통하여 클래스 선언과 테이블 생성을 수행할 수 있습니다. 본 포스팅에서는 장고의 모델에 대해 우선 살펴보겠습니다.

```python
from django.db import models

#SampleModel이라는 이름을 가진 클래스 선언
class SampleModel(models.Model):
    str_attr = models.CharField(
        max_length=32, help_text="max length = 32",
        default="default", db_column="str_column"
    )
    int_attr = models.IntegerField(default=0, db_column="int_column")
    datetime_attr = models.DateTimeField(auto_now_add=True)

#tb_sample이라는 이름의 테이블 생성
class Meta:
    db_table = "tb_sample"
```

#### 클래스를 통한 모델 제어

```python
from django.db import models
from django.core import validators 

class Student(models.Model):

    name = models.CharField(max_length=128, help_text="student name")
    age = models.PositiveIntegerField(help_text="age", default=0)
    phone = models.CharField(
        max_length=32, help_text="phone number",
        validators=[validators.RegexValidator(regex=r"\d{2, 3}-\d{3, 4}-\d{4}")]
    )
    student_ID = models.CharField(max_lenth=64, help_text="student ID")

    created_at = models.DateTimeField(auto_now_add=True, help_text="created time")
    modified_at = models.DateTimeField(auto_now=True, help_text="modified time")

class Meta:
    abstract=False #추상 모델 취급 여부
    managed=True #DB 마이그레이션 여부
    proxy=False #하나의 테이블을 다수의 모델로 분리하여 표현할 지 여부

    db_table = "tb_student"
    get_latest_by = ("created_at", "name")
    ordering = ("-modified_at","name")

    indexes = (
        models.Index(fields=("modified_at", ), name="student_modified_at_idx"),
        models.Index(fields=("name", "age"), name="name_age_composite_idx"),   
    )

    constraints=(
        models.CheckConstraint(check=Q(age__gte=140), name="constraint_abnormal_age"),
        models.UniqueConstraint(fields=("phone",), name="constraint_unique_phone"),
    )
```

##### abstract 
- default = False
- 해당 모델을 추상 모델로 취급할지의 여부
    - 추상 모델로 취급될 경우, DB 마이그레이션 수행 시 제외됨 (즉, DB에 테이블로 반영되지 않음, db_table을 지정할 수 없음)

##### managed 
- default = False
- 해당 값이 False 인 경우, 해당 모델은 DB 마이그레이션에서 제외 (즉, 해당 모델은 장고가 관리하지 않는 것임)
- 이미 생성된 DB 테이블에 매핑하고 싶거나, 장고가 특정 테이블을 수정하는 것을 방지하고자 할 때 사용
- 장고 모델은 데이터베이스 테이블에 매핑할 때 주로 사용되나, 모델과 테이블이 1:1 매핑인 것은 아님 

##### proxy
- default = False
- 하나의 테이블을 2개 이상의 모델로 나눠서 표현하고 싶을 때 사용
- 예시

    DB에는 product라는 테이블에 grocery product, furniture product의 정보가 존재하는 상황에서, grocery product와 furniture product를 각각 관리할 수 있도록 2가지 proxy 모델을 새로 선언하여 매핑

    ```python
    class Product(models.Model):
        class ProductType(TextChoices):
            GROCERY="grocery", "식료품"
            FURNITURE="funiture", "가구"

        name = models.CharField(max_length=128, help_text="상품명")
        price = models.IntegerField(help_text="상품 가격")
        created_at = models.DateTimeField(auto_now_add=True)
        product_type = models.CharField(choices=ProductType.choices, max_length=32)
        store = models.ForeignKey(to="Store", on_delete=models.CASCADE, help_text="판매 가게")

    class GroceryProductManager(models.Manager):
        def get_queryset(self):
            return super().get_queryset().filter(product_type=Product.ProductType.GROCERY)

    class GroceryProduct(Product): #proxy 모델은 반드시 구현이 완료된 모델을 상속받아야 함
        objects = GroceryProductManager()

        class Meta:
            proxy = True

        #grocery product 관련 method들 
        def is_3_days_before_to_expired():
            pass
        
    class FurnitureProductManager(models.Manager):
        def get_queryset(self):
            return super().get_queryset().filter(product_type=Product.ProductType.FURNITURE)

    class FurnitureProduct(Product): #proxy 모델은 반드시 구현이 완료된 모델을 상속받아야 함
        objects = FunitureProductManager()

        class Meta:
            proxy = True 

        #furniture product 관련 method들 
        def is_heavy():
            pass
    ```

##### db_table 
- default: f"{app_label}_{model_name}"; 개발자가 작성한 모델을 기반으로 테이블 이름을 생성 (물론, 개발자가 수정 가능)

    ex) A.models.py 안에 class B(models.Model)이 선언되어 있다면 테이블 이름은 A_B

##### db_table_comment
- 장고 모델 또는 DB 테이블에 대한 주석
    
    ```python
    class Product(models.Model):
        store = models.ForeignKey(
            Store, db_comment = "해당 상품을 가지고 있는 상점"
        )
        name = models.CharField(db_comment="상품명")
    
    class Meta:
        db_table_comment = "상점의 상품"
    ```
- help_text는 장고 모델에서만 볼 수 있으나, db_comment는 DB 테이블 내에서도 동일한 주석 확인 가능

##### get_latest_by
- default: pk
- 장고 쿼리셋의 latest() 호출 시, 가장 최근 값의 기준을 어떤 필드로 사용할 것인지를 명시하는 옵션
- 예시: 최근 데이터의 기준점을 1. 변경 시점, 2. 변경 시점이 동일한 경우, 생성 시점으로 지정
    ```python
    from django.db import models
    from django.core import validators

    class Posting(models.Model):
        author_name = models.CharField(max_length=32, help_text="author name")
        contents = models.CharField(max_length=128, help_text="contents")
        
        created_at = models.DateTimeField(auto_now_add=True, help_text="created time")
        modified_at = models.DateTimeField(auto_now=True, help_text="modified time")

        class Meta:
            db_table = "posting"
            get_latest_by = ("modified_at", "created_at")
    ```

##### ordering 
- default: None 
- 장고 ORM(Object Relational Mapping)으로 데이터 조회 시, 정렬 방법 설정에 이용 
    - 설정값이 지정되지 않았을 경우, DB의 기본 정렬값인 id(pk) 오름차순으로 정렬하여 조회 
       (장고에서는 id와 pk를 동일한 값으로 사용)
- 예시
    ```python
    class Product(models.Model):
        class ProductType(TextChoices):
            GROCERY = "grocery", "식료품"
            FURNITURE = "furniture", "가구"
        
        name = models.CharField(max_length=32, help_text="상품명")
        price = models.IntegerField(help_text="상품 가격")
        created_at = models.DateTimeField(auto_now_add=True, help_text="created time")

        class Meta:
            ordering = ("-created_at", ) #최신 등록 순으로 조회
    ```

##### indexes 
- default: []
- DB 테이블의 인덱스를 지정
- 예시
    ```python
    class Product(models.Model):
        class ProductType(TextChoices):
            GROCERY = "grocery", "식료품"
            FURNITURE = "furniture", "가구"
        
        name = models.CharField(max_length=32, help_text="상품명")
        price = models.IntegerField(help_text="상품 가격")
        created_at = models.DateTimeField(auto_now_add=True, help_text="created time")
        #db_index=True 옵션으로 DB 인덱스 생성 가능 (인덱스 이름 지정 불가 & 원활한 인덱스 정보 명시를 위해 다음과 같이 사용하는 것은 지양)
        #created_at = models.DateTimeField(auto_now_add=True, db_index=True, help_text="created time")

        class Meta:
            indexes = (
                models.Index(fields=["created_at"], name="created_at_idx") #인덱스 이름 지정 가능
                models.Index(fields=["name", "price"], name="name_price_composite_index")
            )
    ```

##### constraints
- default: []
- DB 수준에서 특정 조건을 제한하는 옵션 
- 예시: 상품 가격이 합리적인지, 중복된 상품 이름이 없는지를 확인
    ```python
    class Product(models.Model):
        class ProductType(TextChoices):
            GROCERY = "grocery", "식료품"
            FURNITURE = "furniture", "가구"
        
        name = models.CharField(max_length=32, help_text="상품명")
        price = models.IntegerField(help_text="상품 가격")
        product_type = models.CharField(choices=ProductType.choices, max_length=32)
        
        class Meta:
            constraints = (
                models.CheckConstraint(
                    check=models.Q(price__lte=100_000_000),
                    name="check_unreasonable_price",
                ),
                models.UniqueConstraint(
                    fields=["name", "product_type"],
                    name="unique_product_name",
                ),
            )
    ```