---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[5]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 맵 만들기

- 뱀파이어 서바이벌을 해보셨으면 알겠지만 어느방향으로 가던 맵의 끝이 없고 계속해서 나아갈 수 있습니다.

- 이번 포스팅에서는 무한 맵 이동을 구현해보도록 하겠습니다.


## 타일맵

- Tiles 폴더에 보면 RanTile 이라는 파일이 하나 있습니다. 삭제해줍니다.

- +버튼을 눌러 2D Object - Tiles - Rule Tile을 하나 만들어 줍니다. 이름은 기존과 같이 RanTile로 지정했습니다.

- 타일맵 팔레트를 켜줍니다. Windows - 2D - Tile Palette

- RanTile 파일을 더블클릭하여 인스펙터 창을 열어줍니다. Output을 랜덤으로, Size는 마음대로(영상에서는 10) 지정합니다.

- Size를 지정하면 아래쪽에 스프라이트를 추가할 수 있는데, 여기에 Sprites 폴더에 있는 Tiles를 넣어줍니다.

![image](/images/2023-08-23/capture_1.png)

- 이 10개의 타일이 랜덤하게 선택되어 그려질 것입니다.

- Hierarchy 뷰에서 2D Object - Tilemap - Ractangular를 선택하여 타일맵을 추가한 뒤, 타일 팔레트를 열어 20x20 크기의 맵을 그려줍니다.

![image](/images/2023-08-23/capture_2.png)


## 무한 맵 이동

- 무한 맵 이동 방식은, 현재 만들어둔 Tilemap을 4개정도 사용하여, 플레이어와 거리가 멀어지면 타일맵 위치가 재배치되는식으로 구현합니다.

- Hierarchy 뷰의 타일맵을 선택하고, Tilemap Collider 2D와 Composite Collider 2D 컴포넌트를 추가합니다.

- 그런다음, Tilemap Collider 2D의 Used By Composite를 체크하고, Composite Collider 2D의 Is Trigger 체크를 해줍니다

- 자동으로 추가된 Rigid Body 2D 컴포넌트의 바디타입은 Static으로 설정합니다.

![image](/images/2023-08-23/capture_3.png)



- 그 다음, 태그 두가지를 추가해줍니다. 인스펙터 창의 Tag를 눌러 "Ground", "Area" 라는 이름의 태그를 두개추가합니다. 타일맵의 태그를 Ground로 설정해줍니다.

![image](/images/2023-08-23/capture_4.png)


- Player 오브젝트를 선택하고, Create Empty를 선택하여 자식 오브젝트(Area)를 하나 만들어줍니다.

- Box Collider 2D를 추가하고 그 크기를 20x20으로 설정한 뒤 is Trigger 체크합니다.

![image](/images/2023-08-23/capture_5.png)


### GameManager.cs [게임 매니저]

- GameManager.cs 의 이름으로 스크립트를 만들면 자동으로 톱니바퀴 모양의 아이콘으로 변하게되는데, 다른 스크립트와 차이는 없고 유니티에서 자동으로 아이콘이 바뀌도록 되어있습니다.

- GameManager는 게임에서 필요한 정보들을 편하게 관리하기 위해서 필요한 오브젝트 입니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : MonoBehaviour
{

    public Player player;
    public static GameManager instance;


    private void Awake()
    {
        instance = this;

    }
}

```

- Player 에 관한 정보를 관리하기 위하여 Player 스크립트를 가져옵니다.

- 이 게임매니저도 마찬가지로 다른 스크립트에서 접근하기 쉽도록 메모리에 올려두기 위하여 static 변수를 선언하여, 게임매니저를 메모리에 정적으로 할당해둡니다.


### Reposition.cs [타일맵 재배치 스크립트]

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Reposition : MonoBehaviour
{
    private void OnTriggerExit2D(Collider2D collision)
    {
        if (!collision.CompareTag("Area"))
            return;

        Vector3 playerPos = GameManager.instance.player.transform.position; //플레이어 위치
        Vector3 myPos = transform.position;

        float diffX = Mathf.Abs(playerPos.x - myPos.x);
        float diffY = Mathf.Abs(playerPos.y - myPos.y);

        Vector3 playerDir = GameManager.instance.player.inputVec;
        float dirX = playerDir.x < 0 ? -1 : 1;
        float dirY = playerDir.y < 0 ? -1 : 1;

        switch (transform.tag)
        {
            case "Ground":
                if(diffX > diffY)
                {
                    transform.Translate(Vector3.right * dirX * 40);     //플레이어 가고있는 방향으로 배치
                }
                else if (diffX < diffY)
                {
                    transform.Translate(Vector3.up * dirY * 40);
                }
                break;
            case "Enemy":
                break;
          
        }
    }
}

```

- Reposition.cs 스크립트를 타일맵 컴포넌트로 추가합니다.

- OnTriggerExit2D 함수는 트리거가 체크된 콜라이더에서 나갔을 때 실행되는 함수입니다. 따라서 플레이어가 타일맵 콜라이더를 지나쳐서 나갈 때 실행될 것입니다. 그러므로 우리는 4개의 타일맵을 사용해서, 플레이어가 타일맵 바깥쪽으로 나가려고 할때마다 타일맵을 움직여 계속해서 맵이 이어지는것처럼 보이게 할 것입니다.

- 그러기 위해서는 플레이어와 타일맵의 위치, 플레이어가 향하고있는 방향이 필요할 것이고, 이 정보들을 이용하여 다음 타일맵을 어느위치에 배치할 것인지 정할수 있습니다.


![image](/images/2023-08-23/capture_6.png)


- 타일맵을 복사 붙여넣기 하여 4개의 타일맵을 플레이어 기준 4방향으로 나누어 깔아둡니다. 이제 게임을 시작하여 플레이어가 움직이는 방향쪽으로 타일맵이 재배치 되는지 확인합니다.

![image](/images/2023-08-23/capture_7.gif)
