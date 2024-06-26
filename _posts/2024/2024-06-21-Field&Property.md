---
layer: single
title: "유니티(C#) 필드와 프로퍼티의 차이"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

게임 개발을 공부하던 도중에 다른분들이 만들어놓은것들을 참고하다보니, 필드와 프로퍼티를 많이 사용한다는것을 알게됐다. 대충은 알고있던 개념이지만 내게 익숙하지 않고, 사용방법을 제대로 익혀두기 위해 알아보았다.


## 필드(Field)

- 필드는 클래스나 구조체에 속한 변수로 데이터를 저장한다.
- 필드는 기본적으로 클래스의 상태를 나타내는 데 사용된다.

```c#
public class Player : Monobehavior
{
    public float hp;            // .. 공개 필드
    private float jumpPower;    // .. 비공개 필드
}
```


## 프로퍼티(Property)

- 프로퍼티는 필드에 값을 설정하거나 가져오는 방법을 제공한다.
- 프로퍼티는 get과 set 접근자를 사용하여 필드의 값을 읽거나 변경할 수 있다.

```c#
// 자동구현 프로퍼티
public class MyClass
{
    public int MyProperty { get; set; }
}
```

- 만약 set 접근자에 private 접근 제한자를 붙이게된다면 읽기전용 프로퍼티가 된다.

```c#
// 전통적인 프로퍼티
public class MyClass
{
    private int myProperty;

    public int MyProperty
    {
        get { return myProperty; }
        set { myProperty = value; }
    }
}
```

## 비교

- 필드 : 직접 데이터를 저장. 일반적으로 클래스 내부 로직에 사용
- 프로퍼티 : 필드에 대한 접근방식정의. 유효성 검사, 로깅, 데이터 변환 등 추가로직 포함가능


여기서, 자동구현 프로퍼티는 명시적인 백업 필드를 선언하지 않고도 값을 저장하고 관리할 수 있다.
이것이 가능한 이유는 내부적으로 컴파일러가 필드를 생성하며 프로퍼티를 통해 이 필드를 접근하기 때문이다.

### 다양한 전통적 프로퍼티의 형태

#### 1.기본 프로퍼티

```c#
public class Employee
{
    private int _age;
    
    public int Age
    {
        get { return _age; }
        set { _age = value; }
    }
}
```

- value 키워드는 set 접근자에서 사용되는 내장변수로, 프로퍼티에 할당된 실제 값을 나타낸다.
- 프로퍼티에 값을 할당할 때 value 변수는 그 할당된 값을 갖고있다. 이 값을 사용해 필드에 저장/추가하는 로직을 수행할 수 있다.



#### 2.읽기 전용 프로퍼티

```c#
public class Employee
{
    private int _age;

    public int Age
    {
        get { return _age; }
    }

    public Employee(int age)
    {
        _age = age;
    }
}
```

- 읽기 전용 프로퍼티는 "get 접근자만을 가지며" 값의 수정은 허용하지 않는다. 이는 주로 초기화 이후 값이 변경되지 않아야 할 때 사용한다.


#### 3.쓰기 전용 프로퍼티

```c#
public class Employee
{
    private int _age;

    public int Age
    {
        set { _age = value; }
    }
}
```

- 쓰기 전용 프로퍼티는 "set 접근자만을 가지며" 값의 읽기는 허용하지 않는다. 이는 데이터를 외부로 노출시키지 않으면서 값을 설정할 수 있게한다.

#### 4.유효성 검사가 있는 프로퍼티

```c#
public class Employee
{
    private int _age;

    public int Age
    {
        get { return _age; }
        set
        {
            if(value < 0 || value > 100)
                throw new ArgumentException("나이는 0과 100 사이여야 한다.")
            _age = value;
        }
    }
}
```

- set 접근자 안에서 유효성 검사를 수행하여 잘못된 값이 필드에 할당되는것을 방지할 수 있다.

#### 5.계산된 프로퍼티

```c#
public class Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int Area
    {
        get { return Width * Height; }
    }
}
```

- 계산된 프로퍼티는 다른 필드의 값을 기반으로 계산된 값을 반환한다. 이들은 일반적으로 데이터를 저장하지 않고 필요할 때 값을 계산한다.


## 프로퍼티를 사용하는 이유

- 객체 무결성 보장을 위해 사용한다.

1. 상태의 일관성 : 객체의 속성이나 필드가 항상 예상 가능하고 정의된 규칙 또는 로직에 따라 유지되는 것을 의미
2. 데이터 보호와 유효성 검사 : 객체의 필드나 속성에 잘못된 데이터가 할당되는 것을 방지
3. 캡슐화 : 객체의 내부 데이터를 외부로부터 숨기고 해당 데이터에 접근하거나 변경할 수 있는 메서드만을 외부에 노출시켜 데이터의 무결성 보장


## 필드를 사용하는게 더 좋은 상황

주로 코드의 복잡성이 낮고 성능이 중요한 상황 또는 매우 단순한 로직을 처리할 때는 필드를 사용하는것이 좋다.

1. 단순한 컨테이너 : 주로 데이터를 임시로 보관하거나 다른 계층 간에 데이터를 전달하는 데 사용
2. 성능 최적화 : 프로퍼티 접근자의 추가 오버헤드가 문제가 될 수 있을 때
3. 코드 간결성 : 프로퍼티의 추가적인 구현이 오히려 코드를 복잡하게 만들 때
4. 내부 구현에서의 사용 : 클래스 내부에서만 사용될 때