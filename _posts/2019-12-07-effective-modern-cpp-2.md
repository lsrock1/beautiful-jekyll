---
layout: post
title: 2. C++ auto 연역규칙
subtitle: 
tags: [cpp, modern-cpp, auto]
comments: true
---

auto의 형식 연역은 템플릿 형식 연역과 거의 같다. 템플릿 형식 연역에서는 템플릿 함수에 전달하는 인자의 형태로 템플릿과 매개변수의 형식을 알아낸다.

auto는 그 두가지가 없는데 어떻게 비슷하게 연역할까?

```c++
template<typename T>
void f(ParamType param);

f(expr);
// expr를 이용해 ParamType과 T를 연역한다
```

auto는 연역에서 T의 역할을 하고 형식 지정자가 ParamType의 역할을 한다

```c++
auto x = 27;
// 형식 지정자가 없음

template<typename T>
void f(T param);
// 위와 같은 형식 연역을 하는 템플릿

const auto cx = x;
// 형식 지정자 const

template<typename T>
void f(const T param);
// 위와 같은 형식 연역을 하는 템플릿

const auto& rx = x;
// 형식 지정자 const, &

template<typename T>
void f(const T& param);
// 위와 같은 형식 연역을 하는 템플릿
```

한 가지 경우를 제외하고 템플릿과 동일하게 형식 연역을 한다. auto형식 연역도 템플릿 형식 연역과 동일하게 연역 규칙이 다른 세 가지 경우가 있다

1. 형식 지정자가 참조 형식이지만 보편 참조는 아닌 경우
2. 형식 지정자가 보편 참조인 경우
3. 형식 지정자가 참조가 아닌 경우

작동의 자세한 설명은 템플릿 연역 규칙 스토리를 참조하면 된다. 간단하게 auto 의 형식 연역을 보면

```c++
auto x = 27;
// int
// 형식 지정자가 참조가 아닌 경우(3)

const auto cx = x;
// const int
// 형식 지정자가 참조가 아닌 경우(3)

const auto& rx = x;
// const int&
// 형식 지정자가 참조이지만 보편 참조가 아닌 경우(1)

// 아래는 형식 지정자가 보편 참조인 경우
// 우측 값일 때 다르게 작동(자세한 내용은 템플릿 연역 스토리 참조)

auto&& uref1 = x;
// int&

auto&& uref2 = cx;
// const int&

auto&& uref3 = 27;
// int&&

// 아래는 배열과 함수 이름이 포인터가 되는 경우
const char name[] = "R. N. Briggs";
// const char[13]

auto arr1 = name;
// const char*

auto& arr2 = name;
// const char (&)[13]

void someFunc(int, double);
// void(int, double)

auto func1 = someFunc;
// void (*)(int, double)

auto& func2 = someFunc;
// void (&)(int, double)
```

템플릿 연역과 다르게 작동하는 점이 하나 있는데, 실제로 예전에 연역 규칙을 잘 모르고 코딩할 때 이 형식 연역이 문제를 일으킨 적이 있다.

먼저, 중괄호 초기화를 알아야 한다.
c++ 11부터는 중괄호 초기화를 사용할 수 있다.

```c++
// c++98 부터 가능
int x1 = 11;
int x2(11);

// c++11 부터 가능
int x3 = { 11 };
int x4{ 11 };
```

그런데 변수를 auto로 선언하면 중괄호 초기화의 경우 변수를 std::initializer_list<int>형식으로 연역한다.

```c++
auto x1 = 11;
// int

auto x2(11);
// int

auto x3 = { 11 };
// std::initializer_list<int>

auto x4{ 11 };
// std::initializer_list<int>
```

auto로 선언된 변수의 초기값이 중괄호로 감싸져 있으면, std::initializer_list로 연역한다. 중괄호 안의 값들 형식이 서로 달라서 중괄호 내 형식을 연역할 수 없다면, 컴파일이 거부된다

```c++
auto x5 = { 1, 2, 3.0 };
// int 와 float이 함께 있어서 연역 불가능
// 컴파일 에러
```

즉, 이 중괄호 초기화에 auto를 사용하면 두가지 연역을 하는데
한 가지는 std::initializer_list 형식 연역이고 다른 한 가지는 std::initializer_list<> 내 값들의 형식 연역이다.

이를 통해 std::initializer_list가 std::initializer_list<T> 형태를 가진 템플릿이라는 것을 알 수 있다.

템플릿 형식 연역은 std::initializer_list가 전달될 때 auto 형식 연역과 다르게 작동한다.

```c++
auto x = { 11, 23, 9 };
// std::initializer_list<int>

template<typename T>
void f(T param);

f({ 11, 22, 9 });
// 에러, 연역 불가능
```

템플릿 형식 연역은 std::initializer_list의 인스턴스를 전달하면 연역을 하지 못하고 에러를 뱉는다

```c++
template<typename T>
void f(std::initializer_list<T> param);

f({ 11, 22, 9});
// 작동
```

매개변수 타입을 std::initializer_list를 받을 수 있게 변경하면 형식이 제대로 연역된다.

## C++14 에서 auto 연역

14부터는 함수의 반환과 람다에서 auto를 사용할 수 있는데 이 때는 템플릿 연역 규칙을 사용한다.

```c++
auto createInitList()
{
  return { 1, 2, 3 };
}
// 형식 연역 불가능

std::vector<int> v;
auto resetV = [&v](const auto& newValue) { v = newValue; };

resetV({ 1, 2, 3 });
// 형식 연역 불가능
```

auto와 템플릿을 사용할 때 중괄호 초기화를 한다면 예상과는 다른 형식이 될 수 있으니 주의해야 한다.

위 글은 effective modern c++에 나오는 내용을 정리한 것입니다