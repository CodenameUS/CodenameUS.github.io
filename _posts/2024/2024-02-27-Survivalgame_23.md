---
layer: single
title: "[Unity] 뱀파이어 서바이벌 Like Game[22]"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---

# 오디오 시스템

- BGM, 효과음등의 소리를 넣어보겠습니다.


## Audio Manager

- Audio Manager는 오디오를 관리할 오브젝트로 새로 하나 만들어보겠습니다.

1. Audio Clip : Audio Clip은 오디오 및 사운드 파일 에셋 타입입니다.
2. Audio Source : 에셋인 Audio Clip을 재생시켜주는 컴포넌트입니다.
3. Audio Listener : Audio Source가 내는 소리를 듣는 컴포넌트입니다.

- Add Component를 눌러 Audio 탭에 보면 여러가지 효과음을 낼 수 있는 컴포넌트들이 있습니다.
- Audio Manager 오브젝트에 AudioManager.cs 스크립트를 붙여준 뒤 코드를 작성합니다.


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

    [Header("#SFX")]
    public AudioClip[] sfxClips;
    public float sfxVolume;
    public int channels;        // .. 다량의 효과음을 내기위한 채널 개수 변수
    AudioSource[] sfxPlayers;
    int channelIndex;

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

        // .. 효과음 플레이어 초기회
        GameObject sfxObject = new GameObject("SfxPlayer");
        sfxObject.transform.parent = transform;
        sfxPlayers = new AudioSource[channels];     // .. 채널 개수 만큼 AudioSource 생성

        for(int index = 0;index<sfxPlayers.Length;index++)
        {
            sfxPlayers[index] = sfxObject.AddComponent<AudioSource>();
            sfxPlayers[index].playOnAwake = false;
            sfxPlayers[index].volume = sfxVolume;

        }
    }
} 

```

- AudioManager 스크립트에서는 Bgm(배경음)과 Sfx(효과음)을 관리합니다.
- 게임을 실행하면 Init 함수에서 BgmPlayer, SfxPlayer 라는 오디오 플레이어를 만들고 여러가지 속성들을 초기화합니다.

![image](/images/2024/2024-02-27/capture_1.png)

- 코드를 작성했다면 저장한뒤에 프로젝트폴더의 Audio에 있는 여러가지 효과음들을 적용합니다.


## 효과음 시스템

- 준비는 끝났으니 실제로 게임에서 행동들을 할 때 소리가 나도록합니다.

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

        // .. 효과음 플레이어 초기회
        GameObject sfxObject = new GameObject("SfxPlayer");
        sfxObject.transform.parent = transform;
        sfxPlayers = new AudioSource[channels];  // .. 채널 개수 만큼 AudioSource 생성

        for (int index = 0;index<sfxPlayers.Length;index++)
        {
            sfxPlayers[index] = sfxObject.AddComponent<AudioSource>();
            sfxPlayers[index].playOnAwake = false;
            sfxPlayers[index].volume = sfxVolume;

        }
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

- 행동에 맞는 효과음을 실행시키기위해 열거형으로 효과음을 담았습니다.
- PlaySfx 함수는 열거형으로 선언된 Sfx의 Index를 받아 해당 효과음을 실행하도록합니다.
- 이제 이 함수를 다른 스크립트에서 행동할 때 호출하면 됩니다.

### AchiveManager.cs

- 업적달성 시 효과음을 실행합니다.

```c#
// .. 업적 달성 알림
    IEnumerator NoticeRoutine()
    {
        uiNotice.SetActive(true);   // .. UI 활성화

        AudioManager.instance.PlaySfx(AudioManager.Sfx.LevelUp);

        yield return wait;      // .. 5초 후 

        uiNotice.SetActive(false);  // .. UI 비활성화
    }
```

### LevelUp.cs

- 레벨업 시 효과음을 실행합니다.

```c#
public void Show()
    {
        Next();
        rect.localScale = Vector3.one;  // 1,1,1
        GameManager.instance.Stop();
        AudioManager.instance.PlaySfx(AudioManager.Sfx.LevelUp);

    }

    public void Hide()
    {
        rect.localScale = Vector3.zero;
        GameManager.instance.Resume();
        AudioManager.instance.PlaySfx(AudioManager.Sfx.Select);
    }
```

### Weapon.cs

- 공격 시 효과음을 실행합니다.

```c#
// ... Bullet 1 
    void Fire()
    {
        // ... 대상이 없다면
        if (!player.scanner.nearsetTarget)
            return;

        // ... 총알이 나가고자 하는 방향 설정
        Vector3 targetPos = player.scanner.nearsetTarget.position;
        Vector3 dir = targetPos - transform.position;       // ... 크기가 포함된 방향
        dir = dir.normalized;

        Transform bullet = GameManager.instance.pool.Get(prefabId).transform;
        // ... 위치와 회전 결정
        bullet.position = transform.position;
        // ... 지정된 축을 중심으로 목표를 향해 회전
        bullet.rotation = Quaternion.FromToRotation(Vector3.up, dir);
        // ... count는 관통력
        bullet.GetComponent<Bullet>().Init(damage, count , dir);

        AudioManager.instance.PlaySfx(AudioManager.Sfx.Range);

    }
```

### GameManager.cs

- 게임시작, 게임패배, 게임승리 시 효과음을 실행합니다

```c#
public void GameStart(int id)
    {
        playerId = id;
        health = maxHealth;

        player.gameObject.SetActive(true);
        uiLevelUp.Select(playerId % 2);   // .. 기본 무기 지급
        Resume();

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

        AudioManager.instance.PlaySfx(AudioManager.Sfx.Win);
    }
```

### Enemy.cs

- 몬스터 처치시 효과음을 실행합니다.

```c#
// ... 불릿과 충돌 이벤트
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (!collision.CompareTag("Bullet") || !isLive)
            return;

        health -= collision.GetComponent<Bullet>().damage;
        StartCoroutine(KnockBack());

        if (health > 0)
        {
            // ... 체력이 남아있을 경우
            anim.SetTrigger("Hit"); // ... 애니메이션
            AudioManager.instance.PlaySfx(AudioManager.Sfx.Hit);
        }
        else  // ... 죽었을 때
        {
            isLive = false;
            coll.enabled = false;
            rigid.simulated = false;    // ... 리지드바드 비활성화는 simulated
            sprite.sortingOrder = 1;
            anim.SetBool("Dead", true);

            // ... 킬수증가, exp증가
            GameManager.instance.kill++;
            GameManager.instance.GetExp();

            if (GameManager.instance.isLive)
            {
                AudioManager.instance.PlaySfx(AudioManager.Sfx.Dead);
            }
        }
        
    }
```