---
title: 1. Django 시작하기
author: jdi39
date: 2024-10-05
categories: [Blogging, Django]
tags: [Django]
toc: true
description: Django 개요
---

본 포스팅은 '[백엔드 개발을 위한 핸즈온 장고](https://product.kyobobook.co.kr/detail/S000202404727)'을 읽고 공부한 내용을 정리한 것입니다. 

## 장고(Django)

파이썬 풀스택 웹 프레임워크
  - 프로젝트의 각 도메인과 모듈 간 계측이 높은 응집도와 낮은 결합도를 가질 수 있도록 시스템 아키텍쳐를 분리할 수 있음
    - 장고 앱이라는 단위로 도메인을, URL, 모델, 직렬화(serializers), 뷰(views)로 계층을 분리
        - cf. 마이크로 프레임워크는 도메인과 계층 분리에 대해 자유로움 → 도메인과 계층 분리에 대해 개발자가 책임
  - 비즈니스 로직에 대응하는 API 개수가 많을 때 & 다양한 종류의 인프라(RDB, MessageQueue 등)를 사용해야할 때 적절

### 장고 설계 철학([Django Design Philosophies](https://docs.djangoproject.com/en/5.1/misc/design-philosophies/))

1. 낮은 결합도(Loose coupling)  
   각 계층의 모듈은 교체가 용이해야함; 타 모듈에 대한 의존도를 최소화하여 수정이 용이하도록 해야함        
2. 빠른 개발
3. 적은 코드
4. 중복/반복을 최소화
5. 명시적으로 작성 (Explicit is better than implicit)
   특정 메서드를 오버라이딩하여 의도치 않은 동작이 수행되는 것을 방지하고, 유지보수를 용이하게 하기 위해서는 암시적으로 동작하는 코드의 작성을 지양해야함
   - 장고 시그널: 특정 메서드의 오버라이딩이 불가피한 경우 사용할 수 있는 기능, 어떠한 동작이 수행되었을 때 이를 신호로 받아 다른 동작을 수행하도록 해주는 기능   
                  단, 시그널의 과도한 사용은 복잡도를 높일 수 있으므로 원칙을 정해놓고 사용해야함 (ex. 이력을 쌓아야 하는 경우에만 시그널을 사용한다 등)            
6. 일관성
7. 모델 모듈이 도메인 로직을 캡슐화하도록 작성
모델과 연관된 로직들을 모델 내부에서 관리할 수 있도록 캡슐화
8. 효율적인 SQL 수행 
    SQL 수행은 프레임워크 관점에서 무거운 동작이므로 가능한 한 적게 수행 & 내부적으로 최적화 
    프레임워크가 암시적으로 save를 수행하기 보다는 save()를 명시적으로 사용하는 것이 좋음 & 연계된 모든 객체를 고려해야하는 경우에는 select_related() QuerySet 메서드 사용을 권장
    간결한 database API를 지원하며, 필요할 경우에는 raw SQL 사용도 가능
9. 디자인과 기능 구현을 분리한, 확장이 용이한 템플릿 시스템
10. 캐시 프레임워크(Cache framework)
     간결하고, 일관성있으며 확장이 용이한 cache API 지원

### 장고 어드민

고정된 UI에서 간편하게 데이터 제어(CRUD) 가능. 내부 직원이 사용하는 간단한 툴을 만들기에 적합하나 프론트엔드로 확장을 위해서는 템플릿을 오버라이딩해야만함 (복잡한 프론트엔드 작업이 필요한 상황에는 적합하지 않음)

### 장고 관련 라이브러리

- Django REST framework([DRF](https://www.django-rest-framework.org/))
  장고의 템플릿 뷰를 대신하여 API 방식으로 개발하기 위한 다양한 뷰 구현체(APIView, GenericView, ViewSet)를 제공. API로 프론트엔드와 통신할 수 있도록 지원

- [django-filter](https://github.com/carltongibson/django-filter)
  데이터 검색 기능을 쉽게 개발할 수 있도록 지원하는 라이브러리

- [django-extensions](https://github.com/django-extensions/django-extensions)
  깔끔한 로그 작성 등 개발 편의성을 지원하는 라이브러리

- [drf-spectacular](https://github.com/tfranzel/drf-spectacular)
  API 문서 자동화 도구 지원
