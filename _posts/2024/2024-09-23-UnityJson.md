---
layer: single
title: "유니티 Json을 이용한 데이터 저장 기초-1"
categories: Unity
tag: [C#, Unity, project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

- 게임데이터 저장은 게임에서 필수적으로 있어야할 요소중 하나입니다.  

- 데이터를 저장하는 방법은 여러가지있으나, 가장 많이 사용되는 방법중 하나인 Json을 사용한 방법을 알아봤습니다.  


## Json

- Json(JavaScript Object Notation)이란 데이터를 저장 또는 통신할 때 사용되는 Data 교환 형식입니다.  

- Json의 데이터는 키(key)와 값(value) 쌍으로 이루어진 데이터를 저장하며, 데이터 내용이 텍스트로 되어 있어 사람이 알아보기가 매우 쉽습니다.  

![image](/images/2024/2024-09-23/capture_1.PNG)  


- 내가 작성한 클래스(코드)를 어딘가에 Json 파일(Text형식)으로 변환하여 저장해두고, 필요할 때마다 불러와 사용할 수 있는 빠르고 편리한 방법입니다.  


### Json 사용법

- 유니티에서는 기본적으로 Json을 사용할 수 있는 라이브러리를 제공합니다.  

- 먼저, 샘플 스크립트를 하나 작성합니다.  


```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

class Data
{
    public string name;             // 닉네임
    public int level;               // 레벨
    public int coin;                // 보유코인
}

public class Test : MonoBehaviour
{
    // 데이터 객체 생성
    Data player = new Data() { name = "우성", level = 1, coin = 200 };

    void Start()
    {
        // 1. Data to json 
        // json은 string 형태이므로, string 변수에 저장한다. 
        string jsonData = JsonUtility.ToJson(player);   

        // 2. Json to Data
        // json 파일을 불러올 때는 데이터에 맞는 형식(조립도)가 필요하다.
        Data player2 = JsonUtility.FromJson<Data>(jsonData);

        Debug.Log(jsonData);
    }
}

```

1. 저장하고자 하는 데이터들을 클래스로 만들어 묶었습니다.  
1. 데이터 객체를 생성함과 동시에 임의의 데이터를 집어넣었습니다.  
1. 만들어진 데이터는 JsonUtility 클래스의 ToJson 메서드로 Json화 해줍니다.  
1. Json 파일을 불러올 때는 형식에 맞는 데이터타입이 필요합니다.  

![image](/images/2024/2024-09-23/capture_2.PNG)  



- json 파일이 저장될 경로를 지정하려면 아래와 같은 방식으로 지정해야합니다.  

```c#

string jsonData = JsonUtility.ToJson(player);

// 데이터 저장 경로지정
string path = Path.Combine(Application.dataPath, "playerData.json");

// Application.dataPath 는 기본적으로 Assets 폴더
// Assets 폴더 아래의 다른 폴더(ex. Data)를 지정하여 저장하고 싶을때는
string path = Path.Combine(Application.dataPath + "/Data/", "playerData.json");
```

![image](/images/2024/2024-09-23/capture_3.PNG)  


- 이번 포스팅에서는 Json을 사용한 간단한 데이터 저장 및 불러오기를 해보았고, 다음은 실제 게임에 적용 시켜보도록 하겠습니다.  