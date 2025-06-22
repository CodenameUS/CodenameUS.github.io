---
layer: single
title: "디자인패턴 - 싱글톤(Singleton)"
categories: [Unity, DesignPattern]
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 싱글톤패턴

- 싱글톤패턴은 가장 많이 쓰이는 디자인패턴중 하나로, 객체의 인스턴스가 오직 하나만 생성되도록하는 패턴이다.
- 게임에서 Sound, UI, Data등 시작부터 끝까지 일정하게 관리가 되어야할 때 싱글톤패턴을 사용하곤한다.
- 유니티에서는 이런 오브젝트들을 ~Manager 라는 이름으로 만드는것이 관례라고한다.

![image](/images/2024/2024-05-01/capture_1.png)


## 싱글톤패턴의 장점?

- 싱글톤패턴을 사용하여 구현한것과 그렇지 않은것의 차이점은 3가지 정도로 볼 수 있다.

1. 접근성
2. 유일성
3. 존속성

- 각각이 무엇을 의미하는지 알아보자.

### 접근성

- 일반적으로 다른 스크립트의 함수를 사용하려면 어떤 방법을 사용해서 그 스크립트를 찾아 함수를 꺼내 사용한다.
- 하지만 싱글톤으로 작성된 스크립트는 메모리에 정적으로 할당해두어 언제든 필요할때마다 갖다 쓸 수 있다는 장점이있다.
- 아래는 그 예시이다.

![image](/images/2024/2024-05-01/capture_2.png)

![image](/images/2024/2024-05-01/capture_3.png)

- DataManager의 Save 라는 함수를 Player 스크립트에서 바로 사용할 수 있는것을 볼 수 있다.
- 하지만 이렇게 DataManger 인스턴스를 public으로 작성한다면 보안에 취약해질 수 있다.

![image](/images/2024/2024-05-01/capture_4.png)

![image](/images/2024/2024-05-01/capture_5.png)

- public으로된 프로퍼티를 만들어 DataManager의 인스턴스를 반환하도록 한다면 나을것이다.
- 이런식으로 스크립트에서 DataManager의 함수에 사용하고싶을때마다 쉽게 접근이 가능해지는 장점이있다.

### 유일성

- 예를들어 만약 데이터를 관리하는 DataManager가 여러개 존재한다고 생각해보자.
- 그렇다면 사용자가 데이터 Save를 할 때 마다 여러개의 DataManager들이 저장을 반복할것이고, 이는 분명한 오류이다.
- 스크립트간의 충돌을 막기 위하여 같은 기능을 하는 싱글톤패턴 오브젝트는 오직 하나만 존재하도록 해야한다.

![image](/images/2024/2024-05-01/capture_6.png)

- 만약 기존의 instance가 존재하지 않는다면 나를 새로운 instance로 지정하고, 존재한다면 파괴하는 방식을 통해 오직 하나만 존재하도록 한다.

### 존속성

- 유니티는 장면(씬)이 전환될 때 이전에 있던 오브젝트들은 모두 파괴된다.
- 따라서 이전 장면에서 여러 데이터를 관리하던 Manager들을 다음 장면에서도 이어질 수 있도록 해주어야한다.

![image](/images/2024/2024-05-01/capture_7.png)

- DontDestroyOnLoad(this.gameObject); 를 추가하여 씬이 전환되더라도 기존에 존재하던 이 오브젝트를 파괴하지 않도록 해준다.


![image](/images/2024/2024-05-01/capture_8.png)

- 새로운 씬으로 전환되면 이렇게 DataManager가 살아남은것을 볼 수 있다.

## 싱글톤 오브젝트 관리

- 게임의 규모가 커지게되면 이런 Manager들이 많아질것이다. 여러 Manager들을 관리하는 방법은 여러가지가있다.

1. 하나의 그룹으로 만들어 씬전환마다 끌고가기
2. Manager들만 존재하는 하나의 씬을 만들어 게임이 시작될 때 이 씬을 포함시키기
3. 스크립트를 통해 만들기

- 초보자는 아래와 같이 1번의 방법으로 관리하는것이 좋다고한다.

![image](/images/2024/2024-05-01/capture_9.png)


## 제너릭으로 싱글톤 구현하기

- 싱글톤 오브젝트를 만들때마다 코드를 일일히 만들어야하는 귀찮음을 덜기 위해 제너릭을 사용하여 구현하는방법이있다.
- 새로 Singleton 스크립트를 작성한다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;

    public static T Instance
    {
        get
        {
            // .. instance가 존재하지 않는 경우
            if(instance == null)
            {
                // .. 어딘가 존재할수도 있는 인스턴스 찾기
                instance = (T)FindObjectOfType(typeof(T));

                // .. 없다면
                if(instance = null)
                {
                    GameObject obj = new GameObject(typeof(T).Name, typeof(T));
                    instance = obj.GetComponent<T>();
                }
            }
            // .. 존재하는경우
            return instance;
        }
    }

    private void Awake()
    {
        // .. 부모오브젝트나 최상위 오브젝트가 존재할 때
        if(transform.parent != null && transform.root != null)
        {
            // .. 최상위 오브젝트를 파괴하지않음
            DontDestroyOnLoad(this.transform.root.gameObject);
        }
        else    // .. 자신이 최상위일 때
        {
            DontDestroyOnLoad(this.gameObject);
        }
    }
}

```

- 이제 새로 싱글톤 오브젝트를 만들 때 코드를 일일히 작성할 필요없이 아래와같이 사용하면 된다.

![image](/images/2024/2024-05-01/capture_10.png)

![image](/images/2024/2024-05-01/capture_11.png)

## 주의할 점

- 싱글톤패턴은 편리하지만 메모리에 static으로 할당하기때문에 남용하게되면 메모리를 과도하게 사용할 수 있다는 단점이있으므로 사용에 주의해야한다.
