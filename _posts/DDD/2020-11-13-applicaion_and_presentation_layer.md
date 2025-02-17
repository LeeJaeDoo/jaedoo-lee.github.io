---
date: 2020-11-13 15:40:40
layout: post
title: DDD START! 도메인 주도 설계 구현과 핵심 개념 익히기
subtitle: 6. 응용 서비스와 표현 영역
description: 6. 응용 서비스와 표현 영역
image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/ddd_start.jpeg
category: ddd
tags:
  - ddd
  - 책 정리
paginate: true
comments: true
---
# 표현 영역과 응용 영역
도메인이 제 기능을 하려면 사용자와 도메인을 연결해주는 매개체가 필요하다. 즉, 여기서 응용 영역과 표현 영역이 사용자와 도메인을 연결해 주는 매개체 역할을 하게된다.

* 표현 영역(Presentation Layer)
    * 사용자의 요청을 해석한다.
    * URL, 요청 파라미터, 쿠키, 헤더 등을 이용해 사용자가 어떤 기능을 실행하고 싶어 하는지 판별하고 그 기능을 제공하는 응용 서비스를 실행한다.
    * 응용 서비스가 요구하는 형식으로 사용자 요청 파라미터 변환한다.
    
* 응용 영역(Application Layer)
    * 실제 사용자가 원하는 기능을 제공하는 영역이다.
    * 기능을 실행하는 데 필요한 입력값을 메서드 파라미터로 전달받고 실행 결과를 리턴한다.
    * 트랜잭션 처리를 다룬다.
    
응용 서비스의 메서드가 요구하는 파라미터와 표현 영역이 사용자로부터 전달받은 데이터는 형식이 다르기 때문에 `표현 영역은 응용 서비스가 요구하는 형식으로 사용자 요청을 변환`한다.

사용자와의 상호작용은 표현 영역이 처리하기 때문에 `응용 서비스는 표현 영역에 의존하지 않는다.`
# 응용 서비스의 역할
응용 서비스는 사용자의 요청을 처리하기 위해 리포지터리로부터 도메인 객체를 구하고, 도메인 객체를 사용하여 사용자의 요청을 처리한다.

응용 서비스는 주로 도메인 객체 간의 흐름을 제어하는 역할을 하기 때문에 단순한 형태를 갖는다.<br>
새로운 애그리거트를 생성하는 응용 서비스 역시 간단하다.

* ex.

```java
public Result doSomeCreation(CreateSomeReq req) {
    //  1. 데이터 중복 등 데이터가 유효한지 검사한다.
    checkValid(req);

    //  2. 애그리거트를 생성한다.
    SomeAgg newAgg = createSome(req);

    //  3. 리포지터리에 애그리거트를 저장한다.
    someAggRepository.save(newAgg);

    //  4. 결과를 리턴한다.
    return createSuccessResult(newAgg);
}
```

응용 서비스가 도메인 로직을 일부 구현하면 코드 품질에 안 좋은 영향을 줄 수 있다.

트랜잭션 처리도 응용 서비스에서 다룬다. 그 외에 접근 제어와 이벤트 처리도 응용 서비스에서 다룬다.
## 도메인 로직 넣지 않기
도메인 로직을 도메인 영역과 응용 서비스에 분산해서 구현하면 아래와 같은 이유로 코드 품질에 문제가 발생한다.
* 코드의 응집성이 떨어진다.
    * 도메인 데이터와 데이터 조작 로직이 분산되게 되면 그 만큼 여러 영역을 분석해야 한다는 것을 의미한다.
* 여러 응용 서비스에서 동일한 도메인 로직을 구현할 가능성이 높아진다.

# 응용 서비스의 구현

> 응용 서비스는 표현 영역과 도메인 영역을 연결하는 매개체 역할을 한다. - facade와 같은 역할<br>
> 참고 : [https://lktprogrammer.tistory.com/42](https://lktprogrammer.tistory.com/42)

## 응용 서비스의 크기
응용 서비스를 구현 하는 방식은 보통 아래와 같은 두 가지 방식으로 나뉜다.
* 한 응용 서비스 클래스에 해당 도메인의 모든 기능 구현하기
    * 한 도메인과 관련된 기능을 구현한 코드가 한 클래스에 위치하므로, 각 기능에서 동일 로직에 대한 코드 중복을 제거할 수 있다.
    * 한 서비스 클래스의 크기(코드 줄 수)가 커지게 되고, 이는 결국 가독성을 방해한다.
* 구분되는 기능 별로 응용 서비스 클래스를 따로 구현하기
    * 클래스의 개수는 많아지지만 한 클래스에 모두 구현하는 방식과 비교해서 코드 품질을 일정 수준으로 유지하는 데 도움이 된다.
    * 각 클래스별로 필요한 의존 객체만 포함하므로 다른 기능을 구현한 코드에 영향을 받지 않는다.
    * 각 기능마다 동일한 로직을 구현할 경우에는 별도 클래스에 로직을 구현하는 방식을 활용하여 코드가 중복되는 것을 방지할 수 있다.


## 응용 서비스의 인터페이스와 클래스
보통 인터페이스가 필요한 상황은 아래와 같다.
* 구현 클래스가 여러 개인 경우
* 런타임에 구현 객체를 교체해야 하는 경우

그런데 보통 응용 서비스에서는 위와 같은 경우가 적용되는 경우가 매우 드물다.<br>
따라서, 인터페이스가 명확하게 필요하기 전까지 응용 서비스에 대한 인터페이스를 작성하는 것은 좋은 설계라고 볼 수 없다.

TDD를 즐겨 활용하게 되면 응용 서비스 단을 인터페이스로 작성하게 될텐데 이것도 Mockito와 같은 테스트 도구를 활용하여 테스트용 가짜 객체를 만들 수 있게 되므로, 응용 서비스에 대한 인터페이스 없이 표현 영역을 테스트 할 수 있다.<br>
따라서, 응용 서비스에 대한 인터페이스의 필요성이 약화된다.
## 메서드 파라미터와 값 리턴
응용 서비스가 제공하는 메서드는 도메인을 이용해서 사용자가 요구한 기능을 실행하는 데 필요한 값을 파라미터를 통해 전달받아야 한다.<br>
전달 받는 방식은 다음과 같은 방식이 있다.
* 필요한 각 값을 개별 파라미터로 전달
* 값 전달을 위한 별도 데이터 클래스를 만들어 전달

응용 서비스는 데이터 클래스를 파라미터로 전달받고 필요한 데이터를 추출해서 필요한 기능을 구현하면 된다. 스프링 MVC와 같은 웹 프레임워크는 웹 요청 파라미터를 자바 객체로 변환해 주는 기능을 제공하므로 응용 서비스에 데이터로 전달할 요청 파라미터가 두 개 이상 존재하면 데이터 전달을 위한 별도 클래스를 사용하는 것이 편리하다.

응용 서비스는 표현 영역에서 필요한 데이터만 리턴하는 것이 기능 실행 로직의 응집도를 높이는 확실한 방법이다.
## 표현 영역에 의존하지 않기
응용 서비스의 파라미터 타입을 결정할 때 주의할 점은 표현 영역과 관련된 타입을 사용하면 안된다.(ex. HttpServletRequest, HttpSession과 같은 표현 영역에 해당하는 파라미터가 응용 서비스에 전달되면 안된다.)<br>
이와 같은 응용 서비스에서 표현 영역에 대한 의존이 발생하면 응용 서비스만 단독으로 테스트하기 어려워지기 떄문이다.
## 트랜잭션 처리
트랜잭션 관리는 응용 서비스의 중요한 역할 중 하나다.
## 도메인 이벤트 처리
응용 서비스의 중요한 역할 중 하나로 도메인 영역에서 발생시킨 이벤트를 처리하는 역할이 있다. 여기서 이벤트란 도메인에서 발생한 상태 변경을 의미하여, '암호 변경됨', '주문 취소함'과 같은 것이 이벤트가 될 수 있다.

도메인에서 이벤트를 발생시키면 그 이벤트를 받아서 처리할 코드가 필요한데, 그 역할을 하는 것이 바로 응용 서비스다.

이벤트 처리 코드를 활용하는 이유는 아래와 같다.
* 코드가 다소 복잡해지지만, 도메인 간의 의존성이나 외부 시스템에 대한 의존을 낮춰주는 장점이 있다.
* 시스템을 확장하는 데에 이벤트가 핵심 역할을 수행하게 된다.

# 표현 영역
표현 영역의 책임은 다음과 같다.
* 사용자가 시스템을 사용할 수 있는 (화면) 흐름을 제공하고 제어한다.
* 사용자의 요청을 알맞은 응용 서비스에 전달하고 결과를 사용자에게 제공한다.
* 사용자의 세션을 관리한다.
* 응용 서비스의 실행 결과를 사용자에게 알맞은 형식으로 제공한다. 또는 사용자 요청 파라미터 형식을 응용 서비스가 요구하는 형식으로 파라미터를 변형한다.

# 값 검증
값 검증은 표현 영역과 응용 서비스 두 곳에서 모두 수행할 수 있다. 원칙적으로 모든 값에 대한 검증은 응용 서비스에서 처리한다.<br>
그런데, 표현 영역은 잘못된 값이 존재하면 이를 사용자에게 알려주고 값을 다시 입력받아야 한다. 이 과정에서 다소 번잡한 코드를 작성해야 된다.

응용 서비스에서 각 값에 대한 검증을 할때 문제점은 사용자가 입력 한 값들에 대해 하나하나 검증하게 되면서 불편함을 초래한다. 스프링의 validation 인터페이스를 활용하면 이 문제를 해결할 수 있다.

표현 영역에서 웬만한 필수 값과 값의 형식을 검증하게 되면 실질적으로 응용 서비스는 논리적 오류만 체크하면 된다. 즉, 같은 값 검사를 표현 영역과 응용 서비스에서 중복해서 할 필요가 없게 된다.<br>
따라서, 응용 서비스를 사용하는 표현 영역 코드가 한 곳이면 구현의 편리함을 위해 아래와 같이 역할을 나누어 검증을 수행할 수 있다.
* 표현 영역 : 필수 값, 값의 형식, 범위 등을 검증
* 응용 서비스 : 데이터의 존재 유무와 같은 논리적 오류를 검증.

# 권한 검사
권한 검사는 보통 아래와 같은 3곳에서 수행할 수 있다.
* 표현 영역
    * 사용자의 인증 여부
* 응용 서비스
* 도메인

개별 도메인 단위로 권한 검사를 해야된다면, 다소 구현이 복잡해진다.<br>
도메인 객체 수준의 권한 검사 로직은 도메인별로 다르므로 도메인에 맞게 보안 프레임워크를 확장하려면 프레임워크 자체에 대한 이해가 높아야 한다. 그렇지 않다면, 프레임워크를 사용하는 대신 도메인에 맞는 권한 검사 기능을 직접 구현하는 것이 코드 유지보수에 유리할 수 있다.
# 조회 전용 기능과 응용 서비스
응용 서비스가 사용자 요청 기능을 실행하는 데 별다른 기여를 하지 못한다면 굳이 서비스를 만들 필요가 없다.<br>
예를 들어, 단순히 추가 로직 없이 `DAO`를 활용한 조회만 해오는 기능이 필요하다면 굳이 서비스를 만들 필요 없이 표현 영역에서 바로 조회 전용 기능을 사용해도 된다.
