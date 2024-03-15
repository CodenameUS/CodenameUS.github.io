---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[24]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# BGM 추가 및 로직보완

- 저번에 효과음을 추가한것에 이어서 BGM도 추가해보겠습니다.
- 에셋스토어에서 무료음악을 다운로드하여 프로젝트에 임포트해서 사용했습니다.

## BGM 추가하기

- 먼저 Hierarchy에 AudioManager를 선택하고, 비어있는 BGM부분에 임포트한 BGM을 넣어줍니다.

![image](/images/2024/2024-03-15/capture_1.png)

### AudioManager.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AudioManager : MonoBehaviour
{
    public static AudioManager instance;

    [Header("#BGM")]
    public AudioClip bgmClip;
    public float bgmVolume;
    AudioSource bgmPlayer;
    AudioHighPassFilter bgmEffect;

    [Header("#SFX")]
    public AudioClip[] sfxClips;
    public float sfxVolume;
    public int channels;        // .. 다량의 효과음을 내기위한 채널 개수 변수
    AudioSource[] sfxPlayers;
    int channelIndex;

    public enum Sfx { Dead, Hit, LevelUp=3, Lose, Melee, Range=7, Select, Win}

    void Awake()
    {
        instance = this;
        Init();
    }

    void Init()
    {
        // .. 배경음 플레이어 초기화
        GameObject bgmObject = new GameObject("BgmPlayer");
        bgmObject.transform.parent = transform;
        bgmPlayer = bgmObject.AddComponent<AudioSource>();
        bgmPlayer.playOnAwake = false;
        bgmPlayer.loop = true;
        bgmPlayer.volume = bgmVolume;
        bgmPlayer.clip = bgmClip;
        bgmEffect = Camera.main.GetComponent<AudioHighPassFilter>();

        // .. 효과음 플레이어 초기회
        GameObject sfxObject = new GameObject("SfxPlayer");
        sfxObject.transform.parent = transform;
        sfxPlayers = new AudioSource[channels];  // .. 채널 개수 만큼 AudioSource 생성

        for (int index = 0;index<sfxPlayers.Length;index++)
        {
            sfxPlayers[index] = sfxObject.AddComponent<AudioSource>();
            sfxPlayers[index].playOnAwake = false;
            sfxPlayers[index].bypassListenerEffects = true;
            sfxPlayers[index].volume = sfxVolume;

        }
    }

    public void PlayBgm(bool isPlay)
    {
        if(isPlay)
        {
            bgmPlayer.Play();
        }
        else
        {
            bgmPlayer.Stop();
        }
    }
    public void EffectBgm(bool isPlay)
    {
        bgmEffect.enabled = isPlay;
    }

    public void PlaySfx(Sfx sfx)
    {
        for(int index = 0;index<sfxPlayers.Length;index++)
        {
            int loopIndex = (index + channelIndex) % sfxPlayers.Length;

            if (sfxPlayers[loopIndex].isPlaying)
                continue;

            int ranIndex = 0;
            if(sfx == Sfx.Hit || sfx == Sfx.Melee)
            {
                ranIndex = Random.Range(0, 2);
            }
            channelIndex = loopIndex;
            sfxPlayers[loopIndex].clip = sfxClips[(int)sfx + ranIndex];
            sfxPlayers[loopIndex].Play();
            break;
        }
       

    }
} 

```

- PlayBgm 함수는 캐릭터를 선택하고 게임이 시작되면 BGM이 흘러나오도록하는 함수입니다.
- 레벨업을 했을 때 능력을 선택하는 화면에서 들리는 bgm 볼륨이 작아지도록 하기위하여 AudioHighpassFilter 컴포넌트를 사용했습니다.

### GameManager.cs

```c#
 public void GameStart(int id)
    {
        playerId = id;
        health = maxHealth;

        player.gameObject.SetActive(true);
        uiLevelUp.Select(playerId % 2);   // .. 기본 무기 지급
        Resume();

        AudioManager.instance.PlayBgm(true);
        AudioManager.instance.PlaySfx(AudioManager.Sfx.Select);
    }

    IEnumerator GameOverRoutine()
    {
        isLive = false;

        // .. Dead Animation을 위한 텀
        yield return new WaitForSeconds(0.5f);

        uiResult.gameObject.SetActive(true);
        uiResult.Lose();
        Stop();

        AudioManager.instance.PlaySfx(AudioManager.Sfx.Lose);
        AudioManager.instance.PlayBgm(false);

    }

    IEnumerator GameVictoryRoutine()
    {
        isLive = false;
        enemyCleaner.SetActive(true);

        // .. Dead Animation을 위한 텀
        yield return new WaitForSeconds(0.5f);

        uiResult.gameObject.SetActive(true);
        uiResult.Win();
        Stop();

        AudioManager.instance.PlayBgm(false);
        AudioManager.instance.PlaySfx(AudioManager.Sfx.Win);
    }

```

- 게임시작, 승리, 패배에 따른 BGM On/Off를 추가하였습니다.

### LevelUp.cs

```c#
public void Show()
    {
        Next();
        rect.localScale = Vector3.one;  // 1,1,1
        GameManager.instance.Stop();
        AudioManager.instance.PlaySfx(AudioManager.Sfx.LevelUp);
        AudioManager.instance.EffectBgm(true);
    }

    public void Hide()
    {
        rect.localScale = Vector3.zero;
        GameManager.instance.Resume();
        AudioManager.instance.PlaySfx(AudioManager.Sfx.Select);
        AudioManager.instance.EffectBgm(false);
    }
```
- 레벨업을 했을 때 UI를 보여주고 사라지게할 때 EffectBgm함수를 호출하여 AudioHighpassFilter로 bgm을 걸러듣도록 했습니다.


## 로직보완

1. 이전에 JUMP키를 눌러 레벨업 테스트를 하기위한 코드를 제거했습니다.
2. 게임승리시 EnemyCleaner로 남아있는 모든적을 죽이던 로직에서 문제가 있음을 확인했습니다.

- EnemyCleaner에 Bullet.cs를 사용하고 있는데, 아래 로직(원거리무기)에서 몬스터와 충돌하면 per(관통력)를 1씩 감소시킵니다. 

```c#
void OnTriggerEnter2D(Collider2D collision)
    {   
        if (!collision.CompareTag("Enemy") || per == -100)
            return;

        per--;

        // ... 불릿이 힘을 다했을 때
        if(per < 0)
        {
            rigid.velocity = Vector2.zero;
            gameObject.SetActive(false);
        }
    }
```
- EnemyCleaner에는 리지드바디가 없으므로 조건문에 걸리면 에러가발생합니다. 따라서 EnemyCleaner의 per을 충분히 크게 설정합니다.


3. 기존의 레벨업에 필요한 요구 경험치량이 너무 많은것같아서 조정했습니다.
