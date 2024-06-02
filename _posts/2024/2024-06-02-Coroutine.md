---
layer: single
title: "유니티에서 코루틴[Coroutine]과 IEnumerator"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


![image](/images/2024/2024-06-03/capture_1.PNG)

# 코루틴(Coroutine)

- 코루틴이란 일련의 작업을 다수의 프레임에 분산하여 동작하도록 하는 기능이다.
- 쉽게말해 여러 작업을 '텀'을 두고 동작시킬 수 있다는 것이다.
 
 
## IEnumerator

- 유니티에서는 코루틴을 사용하기 위해 IEnumerator형을 반환값으로 가지는 함수를 사용하고, yield return 구문을 사용하여 흐름을 제어한다.


```c#
public void Use()
    {
        if(type == attackType.Melee)
        {
            StopCoroutine("Swing");
            StartCoroutine("Swing");
        }
        else if(type == attackType.Range && curAmmo > 0)
        {
            curAmmo--;
            StartCoroutine("Shot");
        }
        
    }

    // .. 근접 공격
    IEnumerator Swing()
    {
        isAttack = true;
        yield return new WaitForSeconds(0.1f);
        meleeArea.enabled = true;               // .. 공격범위 활성화
        trailEffect.enabled = true;             // .. 공격효과 활성화

        yield return new WaitForSeconds(0.3f);
        meleeArea.enabled = false;

        yield return new WaitForSeconds(0.3f);
        trailEffect.enabled = false;
        isAttack = false;
    }
```

- 다음과 같은 예시가 있다고 하자. (변수명은 신경쓰지말자)
- 근접공격을 구현하기위해 코루틴으로 작성된 함수 "Swing"이 있다. 
- 애니메이션 동작에 맞춰서 공격을 구현하기위해 각 0.1초, 0.3초, 0.3초의 텀을 두고 각 로직을 실행한것을 볼 수 있다.
- 또한 코루틴을 실행하기 위해서는 StartCoroutine() 메서드를 사용하고, 코루틴을 정지시키기 위해서는 StopCoroutine() 메서드를 사용하면 된다.

## yield return

- yield return은 코루틴 함수내에서 비동기적인 실행을 위한 구문이다.
- 여러가지 반환을 할 수 있는데,

1. yield return null : 한 프레임뒤에 실행을 재개
2. yield return new WaitForSeconds() : 지정한 시간 후에 실행을 재개
3. yield return new WaitForSecondsRealtime() : Time.timescale 값에 영향을 받지 않고 지정된 시간 후 재개
4. yield return new WaitForFixedUpdate() : 모든 스크립트에서 모든 FixedUpdate가 호출된 후에 재개
5. yield return new WaitForEndOfFrame() : 모든 카메라와 GUI가 렌더링을 완료하고 스크린에 프레임을 표시하기 전에 재개
6. yield return StartCoroutine() : 코루틴을 연결하고 코루틴이 완료된 후에 재개

- 등이 있다. 보통은 2번째 옵션을 많이 사용한다.

## 코루틴 실행 및 정지

- 코루틴을 실행하기 위해서는 위에서와 같이 StartCoroutine() 메서드를 호출하면 된다. 
- 매개변수로는 함수의 이름을 넣어주면되는데, StartCoroutine("Swing") 또는 StartCoroutine(Swing()) 과 같이
편한 방법대로 쓰면 된다.
- 반대로 중지하기 위해서는 StopCoroutine() 메서드를 호출하면 된다.
- 만약 모든 실행중인 코루틴을 중단하고 싶다면 StopAllCoroutines() 메서드를 호출하면 된다.


