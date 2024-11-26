---
layer: single
title: "유니티 어드레서블 에셋(Addressable Asset)"
categories: Unity
tag: [C#, Unity]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---



# Intro

이번 포스팅에서는 게임에서 필요한 리소스를 관리하는 방법에대해서 알아봤습니다.

유니티에서 런타임 로드(게임 실행중의 데이터 로드) 방법에는 크게 세가지가있습니다.

- 리소스 폴더 사용
- 에셋 번들
- 어드레서블 에셋

각 방법에 대해서 정리해봤습니다.

## 리소스 폴더

**리소스 폴더**는 유니티에서 제공하는 리소스 로드 폴더로 리소스 폴더 안 파일경로로 접근하여 사용하는 방식입니다.

![image](/images/2024/2024-11-26/capture_1.PNG) 

Assets 폴더 아래에 Resources 폴더를 생성하여 그 안에 필요한 게임리소스를 집어넣고 로드하는 방식입니다.

이 방법은 사용하기 편하다는 장점이 있지만 단점이 많아 권장하지는 않는 방식이라고합니다.

- 빌드 시 함께 묶이게되므로 apk 사이즈가 커진다.
- 앱 시작 시간이 길어진다.
- 앱이나 폴더 변경시 다시 빌드해야한다.
- 에셋 이름 변경등이 힘들다.

이러한 문제점들을 보완할 수 있는것이 에셋 번들입니다.<br>

## 에셋 번들

에셋 번들은 <u>에셋들을 하나로 묶어 압축 파일을 생성하는 개념</u>으로, 생성된 에셋 번들은 네트워크를 통해 배포하고 런타임에 로드하여 사용됩니다.

- 빌드시 포함되지 않으므로 apk 사이즈를 줄여 앱 시작 시간을 단축시킬 수 있다.
- 앱 설치 시 콘텐츠를 분리할 수 있어 라이브 앱에 콘텐츠 업데이트를 제공할 때 사용한다.

![image](/images/2024/2024-11-26/capture_2.PNG) 

하지만 에셋 번들을 활용하는데에도 단점이 있습니다.

- 번들의 종속성 문제 : 같은 에셋일지라도 여러 번들에 묶여있다면 번들의 갯수만큼의 에셋으로 인식
- 따라서 중복된 에셋이 발생하여 메모리 관리가 어려워질수도
- 코드 작성이 쉽지않음

따라서 리소스 폴더의 장점(다소 편리함) + 에셋 번들(빌드와 분리)를 가진 시스템이 바로 어드레서블 에셋 시스템입니다.


## 어드레서블 에셋

어드레서블 에셋이란 말그대로 에셋에 어드레스가 할당된 에셋입니다. 어드레스를 이용해 에셋 위치와는 상관없이 참조가 가능합니다.

어드레서블 에셋을 사용한다면 여러 장점들이 있습니다.

- 에셋 빌드와 배포의 단순화
- 효율적인 에셋 관리가 가능 : 종속성을 파악가능하고 메모리 로드 및 언로드 현황을 볼 수 있다.
- 에셋 번들처럼 카테고리로 분류가 가능하며 각 카테고리는 하나의 그룹으로 묶이게 된다.
- 비교적 간단한 코드
- 대규모 프로젝트에서 메모리를 효율적으로 관리가능하다.

### 어드레서블 에셋 사용방법

어드레서블 에셋을 사용하기위해서는 우선 패키지매니저에서 어드레서블 패키지를 설치해야합니다.

![image](/images/2024/2024-11-26/capture_3.PNG) 

<br>
패키지 설치를 하고나서 에셋을 눌러보면 아래와같이 Addressable 토클 버튼이 생깁니다.

![image](/images/2024/2024-11-26/capture_4.PNG) 

<br>
기본값으로 현재 에셋의 경로로 설정되어있으며 수정이 가능하다.

Addressable 버튼을 활성화하고 Windows > Asset Management > Addressables > Groups를 확인해보면

![image](/images/2024/2024-11-26/capture_5.PNG) 

<br>
아래와 같은 창을 볼 수 있습니다.

![image](/images/2024/2024-11-26/capture_6.PNG) 

<br>
Addressable 에셋으로 설정하면 기본적으로 Default Local Group으로 속하게 되는데, 빈공간에 마우스 우클릭을 통해
그룹 추가가 가능합니다.

그룹은 카테고리로 생각하면됩니다. 중요한것은 그룹내의 하나의 리소스만 사용하려해도 해당 그룹 내 번들을 전부 다운로드받아야만 사용이 가능하다는것입니다. 

저는 Item Sprites Group 그룹에 두가지 포션아이템의 아이콘을 추가했습니다.

<br>
여기에 추가한 애셋을 로드해서 사용하기 위해서는 스크립트에 다음 코드를 추가해주어야한다.

```c#
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;
```

에셋은 서버에 올려서 사용할수도있지만 저는 에디터 내에서 폴더를 하나 만들어 그안에 저장하여 사용했습니다.

```c#
public class ResourceManager : MonoBehaviour
{
    public void LoadAsset(object key)
    {
        try
        {
            Addressables.LoadAssetAsync<GameObject>(key).Completed += (op) =>
            {
                // 로드 성공 시
                if (((AsyncOperationHandle<GameObject>)op).Status == AsyncOperationStatus.Succeeded)
                    return;
                
                // 로드 실패 또는 후속 처리
                
            };
        }
        catch(Exception e) { Debug.LogError(e.Message);  }
    }

}
```

<br>

- LoadAsset 함수는 특정 key를 사용하여 에셋을 로드합니다. 이때 key는 Addressables에 등록된 에셋의 키값으로,
문자열, 정수 등 다양한 데이터유형이 가능합니다. 

- LoadAssetAsync<T>(key) 메서드는 비동기로 에셋을 로드합니다. <T>는 로드할 에셋의 타입응로 여기서는 GameObject입니다.
    - Addressables에 등록된 에셋의 키를 사용하여 해당 에셋을 로드합니다.

- Completed 이벤트는 에셋 로드작업이 완료되었을 때 실행됩니다.
    - op는 AsyncOperationHandle<T> 타입으로 작업의 결과를 포함합니다.

- AsyncOperationHandle<T> 객체의 Status 속성을 확인하여 작업이 성공했는지 체크합니다.
    - AsyncOperationStatus.Succeeded : 에셋 로드가 성공했음을 나타냅니다.

