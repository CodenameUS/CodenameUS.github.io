---
layer: single
title: "C++ 순열과 조합 Permutation, Combination"
categories: Algorithm
tag: [C++, Algorithm]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# Intro

"코딩테스트" 라는것을 공부한지도 이제 두달이 넘어가고, 풀었던 문제도 100문제는 족히 넘어보인다.  

이제는 낯설었던 프로그래밍언어와 어느정도 친숙해졌고, 문제를 보면 어떤방향으로 나아가야할 지도 

보이는것같다.  


![image](/images/2024/2024-10-03/capture_4.PNG)  



오늘 정리할내용은 순열(Permutation)과 조합(Combination)이다. 수학은 고등학생때를 마지막으로 거의 공부한 적 없는것같은데 코딩테스트에서 필요할줄은 몰랐다.         

코딩테스트 문제를 풀다보면 가끔 순열과 조합을 사용해야할 때가 있다. 하지만 이제는 기억조차 나지않을 정도로

공부한 지 오래된 내용이기도하고, 이것을 코드로 어떻게 구현할 지도 모르겠어서 정리해보려고 한다.



## 순열 Permutation

<u>순열</u>이란 "순서"의 개념이 존재하는 조합이다.  

다른말로 "서로 다른 n개의 원소에서 r개를 선택한 후, 이를 나열하는 모든 경우의 수"로 말할 수 있다.   


```c
/*
        예를들어 집합 { 1, 2, 3 } 가 있을 때, 이 3개의 원소로 만들 수 있는 모든 순열의 집합은  

        { 1, 2, 3 }, { 1, 3, 2 }, { 2, 1, 3 }, { 2, 3, 1 }, { 3, 2, 1 }, { 3 , 1 , 2} 

        총 6개의 경우가 나올 수 있다.
*/
```

- 순서가 존재하기 때문에 { 1, 2, 3 }과 { 1, 3, 2 }는 다른 경우의 수가 된다. 




### 순열 구현

먼저, 순서가 존재하고 중복은 허용하지 않는 순열이다.  

재귀호출을 사용하여 구현하였다.  

```c
#include <iostream>

using namespace std;

// { 1,2,3,4 } 에서 3가지를 선택하는 경우의 수 4P3
#define n 4
#define r 3

int pArr[r];                        
int cnt;                            // 총 경우의 수
bool check[n + 1];                  // 중복검사를 위한 배열

// 선택된 각 경우의 수 출력
void PrintArray(int arr[], size_t size)
{
    for (int i = 0;i < size;i++)
    {
        cout << arr[i] << " ";
    }

    cnt++;
    cout << "\n";
}

// 재귀호출을 사용한 순열
void Permutation(int depth) {
    if (depth == r) {
        PrintArray(pArr, r);
        return;
    }

    for (int i = 1; i <= n; i++) {
        if (!check[i]) {
            check[i] = true;
            pArr[depth] = i;
            Permutation(depth + 1);
            check[i] = false;
        }
    }
}

int main() {
    cout << "순열 (순서o, 중복x)\n";
    Permutation(0);
    cout << "총 경우의수 : " << cnt << "개";
}
```



다음으로 중복이 존재하는 순열을 구현해보았다.  

중복검사를 제외한다면 위의 순열 코드와 동일하다.   


```c
#include <iostream>

using namespace std;

// { 1,2,3,4 } 에서 3가지를 선택하는 경우의 수 4P3
#define n 4
#define r 3

int dpArr[r];
int cnt;                            // 총 경우의 수

// 선택된 각 경우의 수 출력
void PrintArray(int arr[], size_t size)
{
    for (int i = 0;i < size;i++)
    {
        cout << arr[i] << " ";
    }

    cnt++;
    cout << "\n";
}

// 재귀호출을 사용한 순열
void DuplicatePermutation(int depth) {
    if (depth == r) {
        PrintArray(dpArr, r);
        return;
    }

    for (int i = 1; i <= n; i++) {
        dpArr[depth] = i;
        DuplicatePermutation(depth + 1);
    }
}

int main() {
    cout << "순열 (순서o, 중복o)\n";
    DuplicatePermutation(0);
    cout << "총 경우의수 : " << cnt << "개";
}
```

## 조합 Combination

<u>조합</u>이란 "순서"의 개념없이 "서로 다른 n개의 원소에서 r개를 선택한 후, 이를 나열하는 모든 경우의 수"로 말할 수 있다.   


```c
/*
        예를들어 집합 { 1, 2, 3 } 가 있을 때, 이 3개의 원소로 만들 수 있는 모든 조합의 집합은  

        { 1, 2, 3 } 

        총 1개의 경우가 나올 수 있다.
*/
```

- 순서가 존재하지 않기 때문에 { 1, 2, 3 } , { 1, 3, 2 } , { 2, 1, 3 }... 등 모두 같은 경우가 된다.



## 조합 구현

먼저 순서가 없으며 중복을 허용하지 않는 조합이다.  

마찬가지로 재귀호출을 사용하여 구현했으며, 반복문을 돌며 모든 경우에 대해 선택하는것은 같으나  

반복문의 시작 값은 이전에 선택한 값 + 1 이 된다.


```c
#include <iostream>

using namespace std;

// { 1,2,3,4 } 에서 3가지를 선택하는 경우의 수 4C3
#define n 4
#define r 3

int cArr[r];
int next;
int cnt;                            // 총 경우의 수

// 선택된 각 경우의 수 출력
void PrintArray(int arr[], size_t size)
{
    for (int i = 0;i < size;i++)
    {
        cout << arr[i] << " ";
    }

    cnt++;
    cout << "\n";
}

// 재귀호출을 사용한 조합
void Combination(int depth, int next) {
    if (depth == r) {
        PrintArray(cArr, r);
        return;
    }

    for (int i = next; i <= n; i++) {
        cArr[depth] = i;
        Combination(depth + 1, i);
    }
}

int main() {
    cout << "조합 (순서o, 중복x)\n";
    Combination(0, 1);
    cout << "총 경우의수 : " << cnt << "개";
}
```


다음으로 중복을 허용하는 조합이다.  

```c
#include <iostream>

using namespace std;

// { 1,2,3,4 } 에서 3가지를 선택하는 경우의 수 4C3
#define n 4
#define r 3

int cArr[r];
int next;
int cnt;                            // 총 경우의 수

// 선택된 각 경우의 수 출력
void PrintArray(int arr[], size_t size)
{
    for (int i = 0;i < size;i++)
    {
        cout << arr[i] << " ";
    }

    cnt++;
    cout << "\n";
}

// 재귀호출을 사용한 조합
void DuplicateCombination(int depth, int next) {
    if (depth == r) {
        PrintArray(cArr, r);
        return;
    }

    for (int i = next; i <= n; i++) {
        cArr[depth] = i;
        DuplicateCombination(depth + 1, i);
    }
}

int main() {
    cout << "조합 (순서o, 중복o)\n";
    DuplicateCombination(0, 1);
    cout << "총 경우의수 : " << cnt << "개";
}
```


## 순열 표준라이브러리 

- C++ 에서 순열을 구성하는 또다른 방법으로, 라이브러리를 사용하는것이다.  

- algorithm 헤더를 포함시키면 사용할 수 있다.  


1. next_permutation : "오름차순의 배열"을 기반
1. prev_permutation : "내림차순의 배열"을 기반

- 매개변수로 (first, last) 범위로 순열을 시작할 범위의 첫번째 주소, 그리고 포함되지 않는 마지막 주소를 넣는다.  

- 매개변수로 iterator를 받기때문에 string 타입의 변수도 순열을 구할 수 있다.  

```c
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
	string s = "4231";
	vector<int> v = { 3, 1, 2 };

	// 오름차순 정렬
	sort(s.begin(), s.end());	
	sort(v.begin(), v.end());

	cout << "***문자열 " << s << "의 순열***\n\n";
	// 문자열 s의 순열
	do {
		cout << s << "\n";
	} while (next_permutation(s.begin(), s.end()));

	cout << "\n\n=======================================\n\n";
	
	cout << "*** 집합 { 1, 2, 3 } 의 순열***\n\n";

	do {
		for (int i = 0;i < v.size();i++)
			cout << v[i] << " ";
		cout << "\n";
	} while (next_permutation(v.begin(), v.end()));


}
```

![image](/images/2024/2024-10-03/capture_5.PNG)  


- next_permutation 함수는 오름차순 정렬이된 데이터를 사용해야한다.  

- next_permutation 함수는 현재보다 더 큰 순열로 재배열할 수 있으면 반복하여 구해내는 구조로, while 문을 사용하여 반복하였다.  

- 만약 데이터가 내림차순으로 정렬되어 있다면 prev_permutation 함수를 사용하면 된다.  