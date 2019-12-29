---
layout: post
title: 3. C++ decltype 연역 규칙
subtitle: 
Tags: [cpp, modern-cpp, decltype]
comments: true
---

연역 규칙 시리즈 중 마지막인 decltype 형식 연역 규칙이다. 한 가지 예외를 제외하면 가장 쉽고 단순하다.

decltype은 declared type의 줄임말인데, 말 그대로 정의된 형식을 그대로 연역한다.

```c++
const int i = 0;
// const int

bool f(const Widget& w);
// decltype(w) == const Widget&
// decltype(f) == bool(bonst Widget&)

struct Point {
  int x, y;
};
// decltype(Point::x) == int
// decltype(Point::y) == int

Widget w;
// decltype(w) == Widget
```

컨테이너(vector, queue…)에서 인덱싱으로(operator[]) 원소를 가져오는 함수를 만드는데 가져오기 전에 유저 인증을 하려고 한다. 그렇다면, 내가만든 커스텀 인덱싱 함수의 선언에서 대상으로 하는 컨테이너의 반환 타입을 반환하게 만들어야 한다.

이 때, decltype을 이용하면 타입을 연역해서 쉽게 함수를 만들 수 있다.

```c++
template<typename Container, typename Index>
auto authAndAcess(Container& c, Index i)
  -> decltype(c[i])
{
  authenticateUser();
  return c[i];
}
```

여기서 auto는 후행 반환 형식을 사용한다는 의미고 형식 연역에 사용되지는 않는다. c와 i는 함수 안에서 사용하는 변수기 때문에 후행 반환 형식이 아니면 정의에서 사용할 수 없다.

.C++ 14에서는 auto(decltype 없이도)로도 후행 반환 형식을 사용할 수 있다. auto가 함수의 반환 정의에 사용되면 템플릿 형식 연역을 따르는데 템플릿 형식 연역에서는 참조성이 사라진다.

```c++
template<typename Container, typename Index>
auto authAndAcess(Container& c, Index i)
{
  authenticateUser();
  return c[i];
}

std::deque<int> d;authAndAccess(d, 5) = 10;
// 컴파일 불가
```

deque<int> 의 operator[] 반환형은 int&인데 템플릿 연역 규칙에 따르면 int로 연역된다. int는 오른값이고 오른값에 10을 넣는건 불가능하기 때문에 컴파일이 불가능하다. 컨테이너의 반환 형식과 정확히 동일하게 연역하기 위해 decltype을 사용하면 된다.

```c++
template<typename Container, typename Index>
decltype(auto) authAndAcess(Container& c, Index i)
{
  authenticateUser();
  return c[i];
}
// c++14 decltype(auto) 사용
```

변수 선언에도 사용할 수 있다

```c++
Widget w;
const Widget& cw = w;
auto myWidget1 = cw;
// Widget

decltype(auto) myWidget2 = cw;
// const Widget&
```

이전 authAndAccess 선언을 보면, 컨테이너를 왼값 참조로 받기 때문에 오른값을 받을 수 없다. 오버라이드 하면 되지만 두개를 관리하기 귀찮다. 보편 참조는 바로 이럴 때 쓰인다.

```c++
template<typename Container, typename Index>
decltype(auto) authAndAcess(Container&& c, Index i)
{
  authenticateUser();
  return std::forward<Container>(c)[i];
}
```

템플릿 매개변수가 보편참조로 선언되었기 때문에 c가 우측값 또는 좌측값으로 들어올 수 있다. 그런데 우측값으로 들어왔을 때 좌측값으로 복사 반환되는 것을 막기 위해 std::forward로 우측값을 그대로 돌려준다(고 한다, 이후에 forward 부분을 다시 정리해서 틀린 부분이 있다면 정정하겠다).

마지막으로 신기한 점은 decltype이 이상하게 작동하는 경우가 있는데 이름이 아닌, 형식이 T인 왼값 표현식에 대해 T& 형식을 연역한다.

```c++
int x = 0;
// decltype(x) == int
// decltype((x)) == int&
```

(x)는 이름은 아닌데 왼값 표현식으로 취급된다. 그래서 int&가 된다. 이런 경우에 예상치 못한 동작을 할 수 있다.

```c++
decltype(auto) f2()
{
  int x = 0;
  return (x);
}
// int& 를 반환
```

책에서도 이런 종류의 코드를 작성하는 것은 미정의 행동으로 직행하는 특급열차에 올라타는 것이라고 말한다. 이상한 괄호를 쓰지 않는 한에서 decltype은 기대한 대로 그 형식을 산출한다.

위 글은 effective modern c++에 나오는 내용을 정리한 것입니다