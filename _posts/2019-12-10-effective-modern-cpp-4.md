---
layout: post
title: 4. 연역 형식 파악
subtitle: 
tags: [cpp, modern-cpp]
comments: true
---

템플릿, auto, decltype을 사용했을 때 실제로 컴파일된 형식을 확인해야 할 경우가 있다.

## IDE 편집기 사용

마우스 커서를 올리면 형식을 표시해 준다. 물론 어느정도 컴파일 가능 한 상태여야 한다.

```c++
const int theAnswer = 42;
auto x = theAnswer;
// int

auto y = &theAnswer;
// const int *
```

## 컴파일러 진단 메세지

보고싶은 형식을 보여주도록 일부러 컴파일 에러를 발생시키면 된다.

```c++
template<typename T>
class TD;
// Type Displayer를 정의 없이 선언만 해둔다

TD<decltype(x)> xType;
TD<decltype(y)> yType;
// x와 y 타입을 보여주는 에러가 발생한다
```

## 실행시점 출력

형식 정보를 printf를 이용해서 표시하는 방법은 실행 때만 사용할 수 있지만 출력 서식을 제어할 수 있다.

```c++
std::cout << typeid(x).name() << '\n';
std::cout << typeid(y).name() << '\n';
```

객체에 typeid를 사용하면 std::type_info 객체를 반환하며, name이라는 멤버 함수가 있고 이 멤버함수는 형식의 이름을 나타내는 C 스타일 문자열(const char*)을 돌려준다는 가정을 하고 있다.

표준에 따르면 std::type_info::name이 의미 있는 정보를 반환한다는 보장이 있는건 아니라고 한다.
예를 들어, GNU 컴파일러는 PKi 라는 정보를 보여주는데
P(pointer) (to) K(const) i(int) 라고 해석하면 된다.

좀 더 복잡한 예를 들면

```c++
template<typename T>
void f(const T& param);

std::vector<Widget> createVec();

const auto vw = createVec();
// const std::vector<Widget>

if (!vw.empty()) {
  f(&vw[0]);
}
// vw[0] == const Widget&
// &vw[0] == const Widget* const&
```

visual studio에서 typeid .name으로 타입을 출력하면 아래처럼 나온다.

T = class Widget const *
param = class Widget const*

T와 param의 형식이 같은 것은 좀 이상하다. param의 타입에는 const와 참조자가 붙어있는데 두 형식이 같은 것은 말이 안된다. 이 문제는 std::type_info::name이 반드시 주어진 형식을 템플릿 함수에 값 전달 매개 변수로서 전달된 것 처럼 취급해야하는 표준 때문에 일어난다.

실제로 param형식은 const Widget* const& 인데 템플릿 함수에 전달 될 때는 const와 참조성이 무시되기 때문에 const Widget* 로 나온 것이다.

## Boost TypeIndex

표준에 포함되지는 않았지만 그에 필적할 정도로 이식성을 보장하는 Boost 라이브러리를 사용할 수도 있다.

```c++
#include <boost/type_index.hpp>

template<typename T>
void f(const T& param)
{
  using std::cout;
  using boost::typeindex::type_id_with_cvr;
  
  cout << "T =    "
       << type_id_with_cvr<T>().pretty_name()
       << '\n';
  
  cout << "param = "
       << type_id_with_cvr<decltype(param)>().pretty_name()
       << '\n';
}
```

참조 한정사들을 그대로 보존해서 형식을 알려주는 std::string 객체를 반환한다.

T = class Widget const *
param = class Widget const * const &

정상적인 결과가 출력된다.

이러저러 방법들이 있지만 최고는 연역규칙을 숙지하는 것이라고 한다.

위 글은 effective modern c++에 나오는 내용을 정리한 것입니다