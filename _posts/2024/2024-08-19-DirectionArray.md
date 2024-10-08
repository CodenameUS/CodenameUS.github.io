---
layer: single
title: "방향을 나타내는 배열"
categories: Algorithm
tag: [C++, Algorithm]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

- 알고리즘 문제를 풀다보면 이동을 구현해야하는 문제가 종종있다.  

- 이동방향을 표현하기위해 switch문이나 if문을 통해 상하좌우를 나타내는 방법이 있지만, 이 방법은 쓸데없이 코드가 늘어나 가독성이 떨어지게된다.  

- 그래서 보통은 방향을 나타내는 배열을 정의하여, 이를 문제에 활용한다.


## 방향배열 정의

- 방향을 어떤식으로 나타낼지는 사람마다 다르다. 아래는 글쓴이가 생각하기 편한방식으로 설명한것이다.  

![image](/images/2024/2024-08-19/capture_1.PNG)      


![image](/images/2024/2024-08-19/capture_2.PNG)      


- 누군가는 방향을 2차원 좌표로 생각하여 좌우를 x(+ -), 상하를 y(+ -)로 정의할 수도 있다.  
- 방향배열이라는것은 문제를 편하게 풀기위한 도구일뿐이지 정해진것은 없다. 따라서 본인이 생각하기 편한쪽으로 정의하여 사용하는것이 제일 좋다!!

