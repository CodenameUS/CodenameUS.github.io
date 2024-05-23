---
layer: single
title: "Serialization[직렬화] & Deserialization[역직렬화]"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---




# 직렬화[Serialization]란?

- Serial의 뜻은 연속적인, 연쇄적인 어떤 것이다.
- 유니티에서 Serialization는 "오브젝트를 연속된 문장형 데이터나 연속된 Byte 데이터로 바꾸는것을 말한다.

![image](/images/2024/2024-05-23/capture_1.PNG)


## 직렬화가 필요한 이유?

- 오브젝트는 메모리에 존재하며 추상적인것에 비해 String 및 Byte 데이터는 드라이브에 저장할수도있고, 통신선을 통해 전송할수도 있다.

- 우리가 게임 엔진을 종료하면 그 프로그램이 사용중이던 메모리의 모든 오브젝트들은 같이 파괴된다. 

- 그러므로 우리는 이것을 저장하고 불러올 필요가 있는데, 이때 사용하는것이 직렬화와 역직렬화다.

- 역으로 이미 존재하는 String 및 Byte 데이터로부터 오브젝트를 생성하는 행위를 <u>역직렬화[Deserialization]</u>라고 한다.

![image](/images/2024/2024-05-23/capture_2.PNG)


- 우리가 에디터에서 편집한다고 생각했던 오브젝트는 직렬화 가능하다는 표시가 붙어있고, 사실 텍스트 파일의 형태로 컴퓨터 어딘가에 저장되어있다.

- 게임 엔진을 다시 실행했을 때, 이 파일들을 역직렬화를 통해서 오브젝트를 다시 생성하는것이다.

![image](/images/2024/2024-05-23/capture_3.PNG)

- 오브젝트는 다양한 텍스트 파일로 변환 가능하며, 유니티에서는 게임 오브젝트의 데이터를 저장하는데 많이 사용된다.


## 직렬화 사용방법

- 직렬화는 앞서 뱀서라이크 게임을 만들 때 사용한 적이 있다.

```c#
// ... 소환 데이터 담당 클래스
[System.Serializable]   // ... 직렬화  
public class SpawnData
{
    public int spriteType;
    public int health;
    public float spawnTime;
    public float speed;
}
```

- 몬스터를 소환하는 클래스인 Spawner에서 스폰 데이터를 관리하기 위하여 SpawnData라는 클래스에 직렬화 옵션을 달아 만든 클래스다.

- 이렇게 작성한 직렬화 클래스는 인스펙터창에서 바로 편집이 가능하며, 필요한 경우 배열의 형태로 선언하여 사용할수도 있다.

```c#
public class Spawner : MonoBehaviour
{
    // ... 스폰데이터를 담을 변수
    public SpawnData[] spawnData;

}
```

- 실제로 직렬화된 데이터를 파일로 저장하는 방법은 다음 글에서 정리해보겠다.


[참조 유튜브(레트로 retro0 님)링크](https://www.youtube.com/watch?v=qrQZOPZmt0w&t=0s)