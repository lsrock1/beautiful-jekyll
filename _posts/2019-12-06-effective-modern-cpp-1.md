---
layout: post
title: 1. C++ 템플릿 연역규칙
tags: [cpp, modern-cpp, template]
comments: true
---

템플릿은 형식을 정하지 않고 컴파일 타임에 함수가 사용되는 모습을 보고 형식을 정한다.

함수 템플릿이 아래와 같은 형태로 존재한다고 하자

```c++
template<typename T>
void f(ParamType param);

f(expr); //호출
```

컴파일러는 f로 형식 T와 ParamType을 연역한다. const같은 수식어로 인해 T와 ParamType의 형식은 달라질 수 있다.

형식을 연역하는 총 세 가지 다른 경우가 있다

- ParamType이 참조 형식이지만 보편 참조는 아닌 경우
- ParamType이 보편 참조인 경우
- ParamType이 참조가 아닌 경우

## ParamType이 참조 형식이지만 보편 참조는 아닌 경우

```c++
template<typename T>
void f(T& param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x);
// T는 int
// param은 int&

f(cx);
// T는 const int
// param은 const int&

f(rx);
// T는 const int
// param은 const int&
```

const로 선언된 값을 템플릿 함수에 전달해도 값을 바꿀 수 없는 const 속성이 유지된다. 이 템플릿 함수의 경우에는 const를 전달해도 안전하다.

f의 매개변수를 T&에서 const T&로 바꿔도 결과는 비슷하다. 단, const가 아닌 변수를 전달하면 param형식에 const를 추가해서 연역한다

```c++
template<typename T>
void f(const T& param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x);
// T는 int
// param은 const int&

f(cx);
// T는 const int
// param은 const int&

f(rx);
// T는 const int
// param은 const int&
```

위의 연역은 매개변수 형식이 포인터로 바뀌어도 동일하게 적용된다

```c++
template<typename T>
void f(T* param);

int x = 27;
const int* rx = &x;

f(&x);
// T는 int
// param은 int*

f(rx);
// T는 const int
// param은 const int*
```

## ParamType이 보편 참조일 경우

```c++
template<typename T>
void f(T&& param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x);
// T는 int&
// param은 int&

f(cx);
// T는 const int&
// param은 const int&

f(rx);
// T는 const int&
// param은 const int&

f(27);
// T는 int
// param은 int&&
```

매개변수 타입이 T&&로 바뀌었다. 오른값 참조와 같은 모습인데 이런 형식의 경우 왼값과 오른값이 전달될 때 각각 다르게 연역된다.

오른값인 27이 전달될 때는 이전의 경우와 비슷한 형태로 T는 int, param은 int&&로 연역되고 왼쪽값이 전달되면 특이하게도 T와 param 둘 다 왼값 참조형식이 된다.

## ParamType이 참조가 아닌 경우

참조가 아니면 그냥 값이 전달되는 것이고 값의 복사가 일어난다.

1. 참조 값이면 참조를 무시한다(& 제거)
2. 참조성을 무시한 후, const이면 const도 무시한다, volatile도 무시한다

```c++
template<typename T>
void f(T param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x);
// T는 int
// param은 int

f(cx);
// T는 int
// param은 int

f(rx);
// T는 int
// param은 int
```

param은 cx나 rx의 복사본이고 원 객체와는 관련이 없다. param의 수정이 cx나 rx에 영향을 미치지 않기 때문에 형식 연역에서 const가 무시된다.

그렇다면 원래 값에 접근할 수 있는 포인터는 어떻게 될까?

```c++
template<typename T>
void f(T param);

const char* const ptr =
  "Fun with pointers";

f(ptr);
// T는 const char*
// param은 const char*
```

첫 번째 const는 포인터가 가리키는 값의 변경을 불가능하게 만들고 두 번째 const는 ptr자체를 변경할 수 없게 한다. param에 전달되는 값은 ptr의 값을 복사한 것이므로 const성이 보전되지 않는다. param은 const 문자열을 가리키는 수정 가능한 포인터가 된다.(포인터가 가리키는 값의 수정은 여전히 불가능)

## 배열 인수

특이한 경우가 있다!

보통 배열의 이름을 포인터로 사용하곤 한다. 실제로 둘은 같지 않고 배열이 배열의 첫 번째 원소를 가리키는 포인터로 붕괴하기 때문에 일어나는 현상이다. 템플릿 함수에 배열의 이름을 전달해보면 어떨까?

```c++
const char name[] = "J. P. Briggs";
// const char[13]

const char* ptrToName = name;
// 배열이 포인터로 붕괴

template<typename T>
void f(T param);

f(name);
// T는 const char*
```

배열 형식의 매개변수는 없다. 배열을 매개변수로 선언 하더라도 포인터 선언으로 취급한다. 신기하게도 참조 매개변수를 사용하는 템플릿은 배열의 실제 형식을 연역한다.

```c++
template<typename T>
void f(T& param);

f(name);
// T는 const char [13]
// param은 const char& [13]
```

이 특성을 이용해 배열에 담긴 원소의 갯수를 컴파일 타임에 연역할 수 있다.

```c++
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept
{
  return N;
}
// constexpr은 함수 호출을 결과를 컴파일 타임에 사용 할 수 있음

int keyVals[] = {1, 3, 7, 9};
int mappedVals[arraySize(keyVals)];
```

## 함수 인수

함수 형식도 함수 포인터로 붕괴할 수 있으며, 이전 배열 형식 연역과 같은 논리가 함수 인수에도 적용된다.

```c++
void someFunc(int, double);

template<typename T>
void f1(T param);

template<typename T>
void f2(T& name);

f1(someFunc);
// param은 함수 포인터
// void (*)(int, double)

f2(someFunc);
// param은 함수 참조
// void (&)(int, double)
```

다음에는 auto 연역규칙에 대해 공부하고 정리할 예정이다

위 글은 effective modern c++에 나오는 내용을 정리한 것입니다