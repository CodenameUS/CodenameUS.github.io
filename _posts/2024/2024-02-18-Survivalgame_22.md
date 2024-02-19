---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[21]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 캐릭터 해금 구현

- 특정조건 달성 시 새로운 캐릭터가 해금되는 것을 구현해보겠습니다.

## 캐릭터 잠금 표현

- 먼저, 캐릭터가 현재 잠겨있는것을 표현한 UI를 만들어보겠습니다.
- 기존의 Character 2를 비활성화 해줍니다.
- 기존의 Character 2를 Ctr+D 하여 복사한 뒤, 이름을 Character 2 (Lock) 으로 변경합니다.
- 해금상태에서 버튼클릭은 필요없으므로 삭제합니다.
- Text Name도 필요없으므로 삭제합니다.

![image](/images/2024/2024-02-18/capture_1.png)

- 위처럼 이미지 색상, 텍스트 내용등을 바꿔줍니다.



## Achive Manager

- Achive Manager 라는것을 만들어서 업적(해금 조건)을 달성했을 때, 캐릭터가 잠금에서 해금이 되도록 할 것입니다.
- Hierarchy view - Create Empty - Achive Manager 라는 이름으로 오브젝트를 만들어 준 뒤, AchiveManager.cs 스크립트를 하나 만들어 붙여줍니다.
- Achive Manager는 앞으로 업적을 관리하는 역할을 할 것입니다.

### AchiveManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using System;
using UnityEngine;

public class AchiveManager : MonoBehaviour
{

    public GameObject[] lockCharacter;      // .. 잠금된 캐릭 목록
    public GameObject[] unlockCharacter;    // .. 해금될 캐릭 목록

    // .. 업적 종류
    enum Achive { UnlockGrape,  UnlockStrawberry }
    Achive[] achives;

    void Awake()
    {
        achives = (Achive[])Enum.GetValues(typeof(Achive));

        // .. 이전의 데이터가 있는지 확인
        if(!PlayerPrefs.HasKey("MyData"))
        {
            Init();
        }
    }


    // .. Device 저장 데이터 초기화 함수
    void Init()
    {
        // .. PlayerPrefs : 간단한 저장 기능 제공하는 클래스
        PlayerPrefs.SetInt("MyData", 1);    // .. SetInt 함수 : key(MyData)와 연결된 int(1) 데이터 저장

        // .. 순차적으로 데이터 저장
        foreach(Achive achive in achives)
        {
            PlayerPrefs.SetInt(achive.ToString(), 0);   // .. 0 : 달성하지 못한것

        }
    }
    void Start()
    {
        UnlockCharacter();
    }

    // .. 캐릭터 해금 함수
    void UnlockCharacter()
    {
        for(int index = 0; index < lockCharacter.Length;index++)
        {
            string achiveName = achives[index].ToString();
            bool isUnlock = PlayerPrefs.GetInt(achiveName) == 1;    // .. 업적을 달성했는 지, 1 : 업적달성(해금)
            lockCharacter[index].SetActive(!isUnlock);              // .. 잠금된 캐릭
            unlockCharacter[index].SetActive(isUnlock);             // .. 해금된 캐릭
        }
    }

    // .. 프레임마다 업적이 달성됐는 지 확인
    void LateUpdate()
    {
        foreach(Achive achive in achives)
        {
            CheckAchive(achive);
        }
    }
    
    // .. 업적 달성 확인 함수
    void CheckAchive(Achive achive)
    {
        bool isAchive = false;

        switch(achive)
        {
            // .. 포도농부 업적 조건 : 킬수 10이상
            case Achive.UnlockGrape:
                isAchive = GameManager.instance.kill >= 10;
                break;
            // .. 딸기농부 업적 조건 : 생존 성공
            case Achive.UnlockStrawberry:
                isAchive = GameManager.instance.gameTime == GameManager.instance.maxGameTime;
                break;
        }

        // .. 업적달성완료 && 해금이 안된 캐릭터일 때 
        if(isAchive && PlayerPrefs.GetInt(achive.ToString()) == 0)
        {
            PlayerPrefs.SetInt(achive.ToString(), 1);   // .. 캐릭터 해금
        }
    }
}

```

- PlayerPrefs 클래스를 사용하여 데이터 저장 및 불러오기를 통해 해금을 구현했습니다.
- 포도, 딸기 농부는 게임 최초 실행 시 잠금된 상태이며 업적달성조건을 만족했을 때 해금이 되도록 했습니다.


## 캐릭터 해금 알림

- 플레이어가 게임플레이중에 업적을 달성하여 캐릭터 해금이 되었을 때 알람이 발생하도록 했습니다.
- 아래와 같은 UI를 만들어서 업적을 달성하면 게임화면 우측상단에 알람이 표시되도록 했습니다.
- Canvas - CreateEmpty - Notice 이름으로 만든 뒤, UnlockGrape, UnlockStrawberry UI를 작성했습니다.
- Notice와 그 아래 오브젝트들은 모두 비활성화 되어있다가 업적달성시에만 표시되어야하므로 비활성화 해둡니다.

### AchiveManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using System;
using UnityEngine;

public class AchiveManager : MonoBehaviour
{

    public GameObject[] lockCharacter;      // .. 잠금된 캐릭 목록
    public GameObject[] unlockCharacter;    // .. 해금될 캐릭 목록
    public GameObject uiNotice;             // .. 해금 알림 UI

    // .. 업적 종류
    enum Achive { UnlockGrape,  UnlockStrawberry }
    Achive[] achives;
    WaitForSecondsRealtime wait;

    void Awake()
    {
        achives = (Achive[])Enum.GetValues(typeof(Achive));
        wait = new WaitForSecondsRealtime(5);

        // .. 이전의 데이터가 있는지 확인
        if(!PlayerPrefs.HasKey("MyData"))
        {
            Init();
        }
    }


    // .. Device 저장 데이터 초기화 함수
    void Init()
    {
        // .. PlayerPrefs : 간단한 저장 기능 제공하는 클래스
        PlayerPrefs.SetInt("MyData", 1);    // .. SetInt 함수 : key(MyData)와 연결된 int(1) 데이터 저장

        // .. 순차적으로 데이터 저장
        foreach(Achive achive in achives)
        {
            PlayerPrefs.SetInt(achive.ToString(), 0);   // .. 0 : 달성하지 못한것

        }
    }
    void Start()
    {
        UnlockCharacter();
    }

    // .. 캐릭터 해금 함수
    void UnlockCharacter()
    {
        for(int index = 0; index < lockCharacter.Length;index++)
        {
            string achiveName = achives[index].ToString();
            bool isUnlock = PlayerPrefs.GetInt(achiveName) == 1;    // .. 업적을 달성했는 지, 1 : 업적달성(해금)
            lockCharacter[index].SetActive(!isUnlock);              // .. 잠금된 캐릭
            unlockCharacter[index].SetActive(isUnlock);             // .. 해금된 캐릭
        }
    }

    // .. 프레임마다 업적이 달성됐는 지 확인
    void LateUpdate()
    {
        foreach(Achive achive in achives)
        {
            CheckAchive(achive);
        }
    }
    
    // .. 업적 달성 확인 함수
    void CheckAchive(Achive achive)
    {
        bool isAchive = false;

        switch(achive)
        {
            // .. 포도농부 업적 조건 : 킬수 10이상
            case Achive.UnlockGrape:
                isAchive = GameManager.instance.kill >= 10;
                break;
            // .. 딸기농부 업적 조건 : 생존 성공
            case Achive.UnlockStrawberry:
                isAchive = GameManager.instance.gameTime == GameManager.instance.maxGameTime;
                break;
        }

        // .. 업적달성완료 && 해금이 안된 캐릭터일 때 
        if(isAchive && PlayerPrefs.GetInt(achive.ToString()) == 0)
        {
            PlayerPrefs.SetInt(achive.ToString(), 1);   // .. 캐릭터 해금

            for(int index = 0;index < uiNotice.transform.childCount; index++)
            {
                bool isActive = index == (int)achive;   // .. 업적 종류에 맞는 UI 활성화
                uiNotice.transform.GetChild(index).gameObject.SetActive(isActive);
            }
            StartCoroutine(NoticeRoutine());
        }
    }

    // .. 업적 달성 알림
    IEnumerator NoticeRoutine()
    {
        uiNotice.SetActive(true);   // .. UI 활성화

        yield return wait;      // .. 5초 후 

        uiNotice.SetActive(false);  // .. UI 비활성화
    }
}

``` 

- 코루틴을 사용하여 알람을 띄운 뒤 5초후에 알람이 사라지도록 했습니다.
- Notice 아래에 UnlockGrape, UnlockStrawberry 오브젝트가 있으므로 이것들도 같이 활성화 해주어야합니다.
