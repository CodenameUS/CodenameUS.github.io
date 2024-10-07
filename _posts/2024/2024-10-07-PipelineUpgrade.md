---
layer: single
title: "유니티 URP 머테리얼 핑크색(마젠타색) 깨짐 간단해결방법"
categories: Unity
tag: [C#, Unity, project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

스토어에서 마음에 드는 에셋을 다운로드받아 내 프로젝트에 추가했더니 

![image](/images/2024/2024-10-07/capture_1.PNG)  

사진처럼 온통 핑크색으로 뒤덮힌 모델을 보게되었다.  




이번 포스팅에서는 자주 발생하는 URP 머테리얼 깨짐을 해결하는 방법을 정리했다.  


## 핑크색으로 나타나는 이유?

프로젝트를 URP로 만들었을 때 발생하는 문제인데,   

결론부터 말하면 Built-in 모델의 재질(머테리얼)이 URP 환경에서 문제가 발생했기때문이다.  

![image](/images/2024/2024-10-07/capture_2.PNG)  


프로젝트에 임포트한 에셋중 Material 폴더의 파일들만 보이는것처럼 깨져있는것을 볼 수 있다.  

따라서 재질을 URP에서 동작할 수 있도록 업그레이드 해주어야한다.


## Built-in To URP

유니티 에디터가 업그레이되면서 기존에 Edit에 있던 업그레이드 기능이, 

Windows - Rendering - RenderPipeline Converter 로 변경되었다.  

![image](/images/2024/2024-10-07/capture_3.PNG)  

빨간색 체크부분의 드롭다운 메뉴를 열어보면, Built-in to URP 항목이 있는데, 그것을 선택한다. 

그런다음, 체크박스 부분을 모두 체크한 뒤, 왼쪽아래의 Initalize Converters를 클릭하면 프로젝트 내의 문제되는 부분들이 자동으로 선택되고, 오른쪽아래의 Convert 버튼을 누르면 알아서 업그레이드해준다.  


![image](/images/2024/2024-10-07/capture_4.PNG)  


간단한 방법으로 해결할 수 있었다.  

![image](/images/2024/2024-10-07/capture_5.PNG)  
