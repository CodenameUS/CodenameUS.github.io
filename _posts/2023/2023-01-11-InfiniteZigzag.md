---
layer: single
title: "[Unity] 간단한 3D 게임 만들기"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---

# 프로젝트 개요
유니티 버전 : 2021.3.15f1

게임명 : Infinite Zigzag

게임 내용 : 플레이어가 마우스 클릭을 통해 자동차 좌우방향을 바꿔가며 오래 살아남는것이 목표.

(게임 예시 사진)
![image](/images/2023-01-11/example.jpg)


## 과정

모바일 환경에서 플레이하는것을 가정하였으므로 유니티 뷰를 1080X1920 으로 맞춰줍니다.

![image](/images/2023-01-11/capture1.png)


Scene 이름도 InfiniteZigzag로 바꿔줍니다.


그다음, Main Camera를 선택하여 Clear Flags를 Solid Color로 바꿔줍니다.(배경 단색)


Background Color는 마음대로 지정합니다.

![image](/images/2023-01-11/capture2.png)



이제 Cube를 하나 생성해봅니다. 카메라(게임뷰)에도 큐브가 생성된것을 확인할 수 있습니다.

![image](/images/2023-01-11/capture3.png)


Cube의 이름을 Platform으로 바꾸고 Scale을 4,1,4로 바꿔 바닥을 만들어봅니다.


Main Camera의 시점을 대각선 위에서 아래를 바라보는 시점으로 적절하게 Position과 Rotation을 바꿔줍니다.

GameObject -> Align with view 를 선택하면 Camera 방향이 Game 뷰에 맞춰지게 됩니다.

그리고 Main Camera -> Projection을 Orthographic로 바꾸어 준뒤 Size를 9정도로 해줍니다.


그러면 대충 이런식으로 구도가 잡히게 됩니다.

![image](/images/2023-01-11/capture4.png)



카메라는 대충 세팅이 되었고 이제 자동차가 움직일 길을 만들어 주겠습니다.



아까 만들어 두었던 Platform을 복사하여 새로운 Platform을 만들고 위치를 Platform의 왼쪽 위에 붙도록 

Position을 Z=3, Scale을 2,1,2로 맞추고 RigidBody를 생성하여 Is Kinematic 체크를 해줍니다.

![image](/images/2023-01-11/capture5.png)



복사한 Platform(1)은 프리팹이 될 녀석입니다. 이름을 PlatformPre로 바꿔주고 Project Assets 폴더에

Prefabs 폴더를 하나 생성한뒤 저장해줍니다.

![image](/images/2023-01-11/capture6.png)


PlatformPre를 3개정도 복사붙여넣기 한 뒤 길이 이어지도록 Z위치를 2씩 띄워줍니다.

![image](/images/2023-01-11/capture7.png)





이제 이 길을 달릴 플레이어를 만들어야 하는데, 저는 Blender로 한번 만들어봤습니다만

![image](/images/2023-01-11/capture8.png)


구글에 무료 Car asset들이 많으니 아무거나 하나 다운로드 받으시면 되겠습니다.




Project - Assets 폴더에 Models 폴더를 하나 생성해주고 다운로드 받은 Car asset을 넣어줍니다.

이름을 Player로 변경하고 Platform위에 올려줍니다.

![image](/images/2023-01-11/capture9.png)




Player에 Rigidbody를 추가하고 BoxCollider를 추가해줍니다.

![image](/images/2023-01-11/capture10.png)


