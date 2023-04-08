---
layer: single
title: "[Unity] 간단한 3D 게임 만들기[2]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


## 자동차가 이동할 블럭 자동으로 생성하기


Hierarchy View에 CreateEmpty -> PlatformSpawner 를 생성합니다.

Scripts 폴더안에 PlatformSpawner.cs 스크립트를 만들고 PlatformSpawner 오브젝트와 연결해줍니다..

![image](/images/2023-04-08/capture_1.png)


```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlatformSpawner : MonoBehaviour
{
    public GameObject platform;

    public Transform lastPlatform;
    Vector3 lastPosition;
    Vector3 newPos;

    bool stop;

    // Start is called before the first frame update
    void Start()
    {
        lastPosition = lastPlatform.position;

        StartCoroutine(SpawnPlatforms());
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    IEnumerator SpawnPlatforms()
    {
        while(!stop)
        {
            GeneratePosition();

            //마지막 플랫폼블럭위치에 newPos만큼 새로운 플랫폼을 생성
            Instantiate(platform, newPos, Quaternion.identity);

            lastPosition = newPos;

            yield return new WaitForSeconds(0.1f);
        }
    }

    void GeneratePosition()
    {
        newPos = lastPosition;

        int rand = Random.Range(0, 2);

        if(rand>0)
        {
            newPos.x += 2.0f;
        }
        else
        {
            newPos.z += 2.0f;
        }
    }
}

```

실행시켜 블럭이 자동으로 생성되는지 확인합니다..




## 카메라

플레이어(자동차)를 따라 움직이는 카메라를 만들어보겠습니다..


CameraController 스크립트를 생성하고 MainCamera에 붙여줍니다.

```c++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraController : MonoBehaviour
{
    public Transform target;
    public float smoothValu;

    Vector3 distance;
    // Start is called before the first frame update
    void Start()
    {
        distance = target.position - transform.position;
    }

    // Update is called once per frame
    void Update()
    {
        Follow();
    }

    //플레이어 따라 움직임
    void Follow()
    {
        //현재 카메라 포지션
        Vector3 currentPos = transform.position;

        //현재 타겟 자동차
        Vector3 targetPos = target.position - distance;

        transform.position = Vector3.Lerp(currentPos, targetPos, smoothValu * Time.deltaTime);
    }
}

```


Main Camera의 CameraController 스크립트 빈칸에 Target->Player, SmoothValu->5를 넣어줍니다.



![image](/images/2023-04-08/capture_2.png)


카메라 시점이 자동차를 따라서 움직이는 것을 확인할 수 있습니다.