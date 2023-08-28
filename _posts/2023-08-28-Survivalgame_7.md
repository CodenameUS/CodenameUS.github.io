---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[6]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# 몬스터 구현

- 몬스터 오브젝트를 추가합니다.

- Sprites 폴더에 있는 Enemy 중 Run0 스프라이트를 가져와서 Hierarchy뷰에 놓습니다.

- 플레이어와 마찬가지로, 그림자를 자식오브젝트로 넣어줍니다.

- Animator 컴포넌트를 추가하여 애니메이션도 추가해줍니다. Enemy 애니메이션은 골드메탈님께서 미리 Animation폴더에 만들어 두셨습니다.

- Animator의 컨트롤러에 해당 Enemy의 애니메이션을 집어넣어줍니다.

- Rigid body 2D도 마찬가지로 추가합니다. 우리는 중력이 필요없으므로 중력값을 0, Freeze Rotation Z를 체크합니다.

- CapsuleCollider 2D를 추가하여, 몬스터 오브젝트의 사이즈에 맞게 조절해 줍니다.

![image](/images/2023-08-28/capture_1.png)


- 만들어진 Enemy 오브젝트를 Ctrl + D 하여 여러종류의 Enemy들도 함께 추가해봅니다. 이때 Sprite Renderer의 Sprite와 Animator의 컨트롤러는 해당 Enemy에 맞는것을 집어넣어주어야합니다.


![image](/images/2023-08-28/capture_2.gif)

- 만약 플레이어가 넘어지는 현상이 발생한다면 Player 오브젝트의 Freeze Rotation을 체크해주어야합니다.

- 플레이어는 몬스터를 밀칠 수 있는 힘이 있어야되므로 Mass값을 15정도로 해줍니다.

## 몬스터 로직

- 몬스터는 플레이어를 따라다니면서 부딪히면 공격당하도록 하게 해야겠죠.

### Enemy.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Enemy : MonoBehaviour
{
    public float speed;         //몬스터 속도
    public Rigidbody2D target;  //타겟

    bool isLive = true;                //생존여부

    Rigidbody2D rigid;
    SpriteRenderer sprite;

    void Awake()
    {
        rigid = GetComponent<Rigidbody2D>();
        sprite = GetComponent<SpriteRenderer>();
    }


    void FixedUpdate()
    {

        if (!isLive)
            return;
       
        Vector2 dirVec = target.position - rigid.position;  //몬스터와 플레이어의 위치차이(방향)
        Vector2 nextVec = dirVec.normalized * speed * Time.fixedDeltaTime;
        rigid.MovePosition(rigid.position + nextVec);
        rigid.velocity = Vector2.zero;
    }

    private void LateUpdate()
    {
        if (!isLive)
            return;
        sprite.flipX = target.position.x < rigid.position.x;    //몬스터가 바라보는 방향설정
    }


}

```

- 몬스터 오브젝트에 이 스크립트를 붙여주고, Speed를 1정도로 설정해줍니다. target은 당연히 Player 오브젝트를 붙여넣어주어야 합니다.

### Reposition.cs

- 플레이어가 몬스터와 멀어지게됐을 때 몬스터가 계속 쫒아오게만 만들면 게임이 시시할 수 있습니다.

- 따라서 몬스터가 멀어지게되면 몬스터를 플레이어 맞은편에 재배치하여 게임이 조금더 어렵도록 합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Reposition : MonoBehaviour
{
    Collider2D coll;

    private void Awake()
    {
        coll = GetComponent<Collider2D>();
    }

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
                if (coll.enabled)   //몬스터가 살아있을 경우
                {
                    //플레이어의 맞은편에서 나타나게하기
                    transform.Translate(playerDir * 20 + new Vector3(Random.Range(-3f, 3f), Random.Range(-3f, 3f),0f));      
                }
                break;
          
        }
    }
}

```

- 마찬가지로 이 스크립트를 Enemy오브젝트에 추가해줍니다.

- 그런다음 Enemy 오브젝트들의 태그를 Enemy로 설정합니다.

- 원활한 테스트를 위하여 Enemy 오브젝트를 Ctrl + D 키를 사용하여 많이 복사해서 게임을 실행해봅니다.

![image](/images/2023-08-28/capture_3.gif)
