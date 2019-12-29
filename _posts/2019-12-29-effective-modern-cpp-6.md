---
layout: post
title: 6. auto가 아니라 명시적 형식 초기화를 사용해야 할 때
subtitle: 
tags: [cpp, modern-cpp, auto]
comments: true
---

책에서는 auto를 사용하는게 명시적 초기화를 사용하는 것 보다 좋다고 한다. auto를 사용할 때 예상치 못한 타입으로 연역되는 경우가 있다. 그러한 경우를 알아보고자 한다.

#### Proxy class가 사용되는 경우

Proxy class는 실제 클래스의 기능을 보조하는데 쓰는 클래스다. 여기서는 bool을 보조해서 효율적으로 bool을 저장하고 사용자가 실제로는 bool을 사용하는 것 처럼 느끼게 한다.

std::vector<T> 의 operator[]는 보통 T&를 돌려준다. 하지만, T가 bool일 경우에 operator[]는 std::vector<bool>::reference를 반환한다.

std::vector<bool>는 bool을 1비트의 압축된 형태로 표현하고 C++는 비트에 대한 참조를 금지한다. 그래서 bool의 참조 대신 std::vector<bool>::reference라는 proxy class를 반환하게 된다.

```c++
std::vector<bool> features = { true, false };
bool output = features[0];
```

위와 같은 코드를 짜면, std::vector<bool>::reference가 bool로 변환된다.(bool& 아님)
정확히 5번째 비트값을 가진 bool로 변환해준다. 

```c++
std::vector<bool> features = { true, false };
auto output = features[0];
```

위와 같은 코드를 짜면 auto에 의해 output의 형식이 연역되기 때문에 실행 결과가 어떻게 될 지 예상할 수 없다. output은 std::vector<bool>::reference의 구현 방식에 따라 다른 값을 가지게 된다. proxy class들은 보통 객체의 수명을 한 문장으로 가정하는 경우가 많다.

```c++
auto someVar = proxy class
```

위와 같은 형태는 최대한 피하는 것이 좋다.

책에서는 auto를 사용하되 아래처럼 명시적으로 초기화 형식을 지정하라고 말한다.

```c++
auto output = static_cast<bool>(features[0]);
```

이런 방법은 형식을 바꾸려는 의도를 드러내는데 매우 유용하게 활용할 수 있다.

```c++
double calcEpsilon();

float result = calcEpsilon();
// double이 float으로 바뀌는지 잘 모름

auto result = static_cast<float>(calcEpsilon());
// float로 바꾸어 저장하겠다는 의도가 드러남
```

요약하자면 proxy class를 사용했을 때 auto로 변수를 선언해서 저장하면 의도와 다른 결과가 나올 수 있다. 그래서 static_cast로 형변환을 해서 의도한 형태로 받아주는게 좋다. 이런 명시적 형변환은 의도를 드러내는데 사용될 수 있다.

위 글은 effective modern c++에 나오는 내용을 정리한 것입니다