---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[1]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 프로젝트 준비

- 프로젝트명 : Survival Game

- 템플릿은 2D(URP)로 만들었습니다.


## 애셋 준비

- 게임을 만드는데 필요한 애셋들은 골드메탈님 블로그에서 무료로 제공하고있습니다.

- 참고 : https://www.goldmetal.co.kr/


![image](/images/2023-08-10/capture_1.png)


- 다운로드 받은 파일을 압축해제한 뒤 유니티에 드래그 드랍해서 임포트 해줍니다.

![image](/images/2023-08-10/capture_2.png)

- 이 애셋들은 개인 학습용, 과제, 상용 프로젝트에도 자유롭게 사용이 가능하다고 하니 참고하시면 됩니다.



## 오브젝트 준비

- 임포트된 패키지파일을 살펴보면 다음과 같은 파일들이 있습니다.

![image](/images/2023-08-10/capture_3.png)


- 그 중에서 Sprite 폴더에 보면 여러 아틀라스들이 있는것을 확인할 수 있습니다. 이 아틀라스들을 잘라서 사용하겠습니다.

- 아틀라스를 자르기 위해서, Sprite Mode를 Multiple로 지정하고, Sprite Editor를 눌러줍니다.

![image](/images/2023-08-10/capture_4.png)


- Slice를 눌러, Grid by Cell Size를 선택하고 Pixel Size는 X : 18 Y : 20으로, Padding은 1, 1로 설정해줍니다.


![image](/images/2023-08-10/capture_5.png)


- 이제 잘라진 스프라이트를 Scene에 추가해 보겠습니다.

![image](/images/2023-08-10/capture_6.png)


## 컴포넌트 추가

- Scene에 추가한 Player 오브젝트에 컴포넌트를 붙여감으로써 우리가 필요한 기능들을 추가할 수 있습니다.

- 먼저, RigidBody 2D 컴포넌트를 추가하여 물리를 적용해보았습니다.

- Rigidbody 2D 컴포넌트를 보면, Gravity Scale 항목이 있습니다. 중력에 관한것으로 뱀파이어서바이벌에서는 중력이 필요없으므로 0으로 설정합니다.

- 그 다음 Capsule Collider 2D 컴포넌트를 추가합니다. 이 컴포넌트는 오브젝트의 충돌을 관리합니다.

- Size가 플레이어와 맞지 않으면 적절하게 설정합니다.


![image](/images/2023-08-10/capture_7.png)


## 플레이어 그림자 추가

- Sprites 폴더에 보면 Props 라는 아틀라스가 있습니다. 그 안에 Shadow 스프라이트가 있습니다. 이를 플레이어에 붙여넣어 자식 오브젝트로 설정합니다.

- Player 레이어의 Order를 1로 설정함으로써 Shadow가 플레이어보다 뒷쪽에 보이도록 할 수 있습니다.


![image](/images/2023-08-10/capture_8.png)


## 배경색 설정

- Main Camera를 선택하고, Camera - Environment의 Background 항목에서 배경색을 지정해줄 수 있습니다.

- RGB 값을 255 255 255로 설정하여 새하얀 배경색으로 설정해보았습니다.

![image](/images/2023-08-10/capture_9.png)
