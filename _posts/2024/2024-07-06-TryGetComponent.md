---
layer: single
title: "유니티(C#) TryGetComponent"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

- TryGetComponent와 GetComponent의 차이를 알고싶어 찾아보았다.  


## TryGetComponent 함수의 원형

```c#
public bool TryGetComponent(Type type, out Component component);

public bool TryGetComponent(out T component);
```

- TryGetComponent 함수는 bool 형의 함수로, 해당 타입의 컴포넌트를 찾았다면 true를 아니면 false를 반환한다.

- 만약 찾았다면 out되는 component에 해당 타입 컴포넌트를 할당해준다.


## GetComponent 와의 차이점

- GetComponent는 그 타입의 컴포넌트를 찾지 못했을 때 memory allocation을 발생시켜 가비지 컬렉션을 야기시킬 수 있다고 한다.  

- 하지만 TryGetComponent는 그렇지 않아 가비지 컬렉션을 걱정할 필요가 없다고 한다.  

- TryGetComponent는 유니티 2019.2 버전부터 사용할 수 있다.
