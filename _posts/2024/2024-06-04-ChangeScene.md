---
layer: single
title: "유니티 씬(Scene) 이동 및 로딩씬 구현"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

- 이 포스팅에서는 다른씬으로 이동하는것과, 그 사이에 로딩씬을 만들어 자연스러운 씬 전환을 구현해보려고합니다.
- 단계별로 차근차근 따라하면 쉽게 이해할 수 있습니다.


## 포탈(Portal) 만들기

- 간단한 포탈을 만들어 플레이어가 포탈에 접근하면 다른 씬으로 전환되도록 합니다.

![image](/images/2024/2024-06-04/capture_1.PNG)

- 하이라키뷰에서 빈 오브젝트를 하나 만들어 Particle System 컴포넌트를 추가합니다.

![image](/images/2024/2024-06-04/capture_2.PNG)

- 포탈을 표현하기 위한 내 입맛대로 파티클 속성을 변경합니다. 아래는 제가 설정한 값입니다.

1. Renderer - Material : DefaultLine
2. Start Life Time : 1, Start Speed : 0, Start Size : 0.2
3. Emission - rate over Time : 200
4. Shape - shape : Donut, Donut Radius : 0.0001, Arc - mode - Loop, Rotation X : 90


- 이 오브젝트 아래에 새로운 빈 오브젝트를 추가하여, Particle System 컴포넌트를 추가한 뒤, 좀 더 생동감 있는 포탈을 표현했습니다.

![image](/images/2024/2024-06-04/capture_3.gif)


- 마지막으로 포탈에 Sphere Collider 컴포넌트를 추가한 뒤, Is Trigger 옵션을 체크합니다.


### 포탈 스크립트

- !! [씬 만들기] 이후의 과정을 먼저 수행한 뒤 스크립트를 작성하시길 바랍니다.
- !! 반드시 플레이어 오브젝트의 태그를 "Player" 로 설정해야합니다.
- 플레이어가 포탈에 접근하면 씬 전환을 위한 충돌 이벤트를 발생 시키도록 했습니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SceneChange : MonoBehaviour
{
    void OnTriggerEnter(Collider other)
    {
        if (!other.CompareTag("Player"))
            return;

        LoadingSceneController.LoadNextScene("Dungeon1");
    }
}
```

- 스크립트 이름은 SceneChange 입니다.
- OnTriggerEnter 함수를 사용하여, Player와 충돌하게되면 이벤트가 발생하도록 했습니다.
- 뒤에서 만들 LoadingSceneController 클래스의 LoadNextScene 메서드를 호출하여 "Dungeon1" 씬을 불러옵니다.

## 씬 만들기

- 포탈이 있는 현재의 씬을 Lobby, 프로젝트 폴더에 Create - Scene 으로 Dungeon과 Loading 이름으로 씬을 만들었습니다.

![image](/images/2024/2024-06-04/capture_4.PNG)

- 유니티 엔진의 File - Build Settings.. 에서 Scene In Build 항목에 만들어둔 씬들을 추가해주어야 합니다.

![image](/images/2024/2024-06-04/capture_5.PNG)

- 그러면 씬 파일의 경로와 함께 오른쪽에 번호가 보이는데, 이 번호는 각 씬의 Index 라고 생각하면 됩니다.
- 나중에 씬을 불러오고자 할 때 씬 경로, 씬 이름, 인덱스로 불러올 수 있습니다.


### Dungeon 씬

- Dungeon 씬을 더블클릭하여 실행해봅니다.
- Dungeon 씬에는 별것없이, Lobby 씬의 바닥 오브젝트만 복사하여 붙여넣었습니다.

![image](/images/2024/2024-06-04/capture_6.PNG)


### Loading 씬

- Loading 씬을 더블클릭하여 실행해봅니다.
- Loading 씬에서는 씬에서 씬으로 전환될 때, 중간 씬을 만들어 사용자에게 자연스러운 씬전환을 연출하기위한 로딩창을 만들어봅니다.

[골드메탈 - 쿼터뷰 3D 액션 에셋 팩 링크](https://assetstore.unity.com/packages/3d/characters/quarter-view-3d-action-assets-pack-188720)

- UI에 사용된 이미지 소스들은 애셋 스토어에서 다운로드 받으실 수 있습니다.

![image](/images/2024/2024-06-04/capture_7.PNG)

- Canvas 설정은 아래와 같이 해주었습니다.

![image](/images/2024/2024-06-04/capture_10.PNG)

- 로딩창 UI는 본인 입맛에 맞게 만들어 주시면 됩니다.

#### 차오르는 로딩바 구현

- 로딩바를 구현하는 방법은 다양합니다. 그 중에서 저는 이미지의 Rect Transform의 Width 길이를 조절하여 로딩 게이지가 차오르는것처럼 구현해봤습니다.

- Loading Bar 및 Loading Gaue Bar 를 아래와 같이 설정했습니다.

![image](/images/2024/2024-06-04/capture_8.PNG)

![image](/images/2024/2024-06-04/capture_9.PNG)

- Anchor 설정과 여백 및 Scale를 조절하여 적당하게 배치해주었습니다.


```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class LoadingSceneController : MonoBehaviour
{
    static string nextScene;
    string[] gameTips = {
        "체력과 마나는 포션으로 보충할 수 있습니다.",
        "더 좋은 장비는 던전클리어에 큰 도움이 됩니다.",
        "플레이어가 사망하면 마을로 돌아갑니다."
    };
                    
    
    [SerializeField] Image  progressBar;
    [SerializeField] Text   tipText; 


    // 다음 씬 불러오기
    public static void LoadNextScene(string sceneName)
    {
        nextScene = sceneName;
        SceneManager.LoadScene("Loading");
    }

    void Start()
    {
        StartCoroutine(LoadSceneProcess());
        ShowRandomTips();
    }

    IEnumerator LoadSceneProcess()
    {
        AsyncOperation op = SceneManager.LoadSceneAsync(nextScene);
        op.allowSceneActivation = false;

        float timer = 0f;

        while(!op.isDone)
        {
            yield return null;

            if(op.progress < 0.9f)
            {
                progressBar.rectTransform.sizeDelta = new Vector2(op.progress * 1920f, 80f);
            }
            else
            {
                // 씬 로딩이 90% 완료되면 완료대기시키고 Fake 로딩주기
                timer += Time.unscaledDeltaTime;
                progressBar.rectTransform.sizeDelta = new Vector2(1728f + timer * 20f, 80f);
                if(progressBar.rectTransform.sizeDelta.x >= 1920f)
                {
                    op.allowSceneActivation = true;
                    yield break;
                }
            }
        }
        
    }

    // 게임팁 랜덤 텍스트 표시
    void ShowRandomTips()
    {
        int ran = Random.Range(0, gameTips.Length);

        tipText.text = "Game Tip. " + gameTips[ran];
    }
}

```

- LoadingSceneController 클래스는 LoadNextScene에서 전환할 씬 이름을 매개변수로 받아 해당 씬으로 전환합니다.

- LoadNextScene은 public static으로 어느 스크립트에서든 호출하여 사용할 수 있도록 했습니다.

- 로딩바를 채우는기능은 코루틴을 사용하여 구현했습니다.

- 씬을 불러오는 진행상황은 AsyncOperation 타입으로 반환하므로, 이를 op 라는 변수에 넣어두고 allowSceneActivation 옵션을 false로 지정합니다. 이것은 씬의 로드가 끝났을 때 자동으로 씬 전환이 되는것을 막아줍니다.

- while문에서는 isDone 옵션을 체크하여 씬 로딩이 끝나기전까지 반복합니다. 

- 각 반복이 끝날 때 마다 yield return null; 을 통해 유니티 엔진에 제어권을 넘겨주어 로딩바가 차오르는것을 표시할 수 있도록 했습니다.

- 씬 로딩 진행상황이 90% 전까지는 진행 상황에 따라 이미지의 Width를 조절하여 왼쪽에서 오른쪽으로 차오르도록 했습니다. 다만, 지금같은 경우에는 씬 로드속도가 매우 빠르므로 이는 로딩창에서 확인하기는 어렵습니다.

- 이후에는 timer 변수를 통해 Fake 로딩을하도록하여 사용자가 로딩씬에서 게임팁 등을 볼 수 있는 여유를 두었습니다. 게이지바가 100% 차게되면 씬 전환이 되도록 했습니다.

- 마지막으로 여러가지 게임팁을 랜덤하게 보여줄 수 있도록 랜덤값을 통해 게임팁 텍스트가 출력되도록 했습니다.


## 실행해보기

![image](/images/2024/2024-06-04/capture_11.gif)
