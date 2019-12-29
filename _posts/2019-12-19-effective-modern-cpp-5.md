---
Layout: post
Title: 5. 명시적 형식 선언보다는 auto를 선호하라
Tags: [cpp, modern-cpp, auto]
comments: tru
---

auto를 사용하면 타자의 양이 줄어들고 정확성 문제와 성능 문제를 방지할 수 있다. auto가 의도하지 않은 형식을 연역할 수 있지만 auto를 포기하고 형식을 명시적으로 지정하는 것은 최대한 피하는 것이 좋다.

## 타자의 양이 줄어든다

```c++
template<typename It>
void dwim(It b, It e)
{
  for (; b != e; ++b) {
    typename std::iterator_traits<It>::value_type
     currValue = *b;
  }
}
```

typename std::iterator_traits<It>::value_type 은 너무 길다. auto 로 간단히 정리할 수 있다.

## 클로저 형식 선언 가능

```c++
auto derefUPLess =
  [](const std::unique_ptr<Widget>& p1,
     const std::unique_ptr<Widget>& p2)
  { return *p1 < *p2; };
```

원래는 이 클로저의 형식을 컴파일러만 알고 있었지만 auto로 연역해서 변수를 선언할 수 있다.

C++14에서는 람다 표현식의 매개변수에도 auto를 적용할 수 있다.

```c++
auto derefUPLess =
  [](const auto& p1,
     const auto& p2)
  { return *p1 < *p2; };
```

물론 이런 함수 포인터를 가리키기 위해 std::function을 사용할 수 있다. std::function은 C++11부터 포함된 호출 가능한 객체를 가리킬 수 있는 표준 라이브러리 템플릿이다.

```c++
std::function<bool(const std::unique_ptr<Widget>&,
                   const std::unique_ptr<Widget>&)>
  derefUPLess = [](const std::unique_ptr<Widget>& p1,
                   const std::unique_ptr<Widget>& p2)
                  { return *p1 < *p2; };
```

위와같은 std::function으로 전에 선언된 클로저를 받을 수 있다.

auto와 std::function은 타이핑 외에도 정말 큰 차이점이 있다. auto는 정확히 클로저와 같은 형식으로 선언된다. 하지만 std::function는 템플릿의 인스턴스고 크기가 시그니처에 대해서 정해져있다. 그런데 고정된 크기가 실제 클로저에 비해 부족할 수 있다. 이 때는 힙 메모리를 사용한다. 결과적으로 auto에 비해 메모리를 많이 사용하고 더 느리다.

## type shortcut과 관련된 문제를 피할 수 있다

```c++
std::vector<int> v;
unsigned sz = v.size();
```

v.size()의 반환 형식은 std::vector<int>::size_type인데 unsigned로 충분하다고 생각하고 위와 같이 작성할 수 있다.

이 코드는 예상치 못한 버그를 일으킬 수 있다. Windows 32비트에서는 unsigned와 std::vector<int>::size_type가 같은 크기인데 64비트 버전에서는 unsigned는 32비트이고 std::vector<int>::size_type는 64비트다. 즉 데이터가 날아갈 수 있다. 그래서 그냥 auto를 쓰면 좋다.

## 형식 불일치 문제를 피할 수 있다

```c++
std::unordered_map<std::string, int> m;
for (const std::pair<std::string, int>& p : m) {
  // something
}
```

위 코드는 언듯 문제가 없어보인다. 하지만 std::unordered_map의 키 부분이 const기 때문에 실제로 pair의 형식은 std::pair<const std::string, int> 가 된다.

컴파일러는 선언된 형식에 맞추기 위해 pair m의 객체를 복사하고 그 객체를 p에 할당해서 문제를 해결한다. 원래 의도는 m의 요소들을 p에서 참조하는 것이지만 실제로는 매우 복잡한 작업이 일어난다. 당연하게도 auto를 쓰면 이런 문제를 해결할 수 있다.

책에서는 명시적 선언 대신 auto를 사용할 것을 최대한 권장한다. auto를 사용하더라도 IDE가 형식 추론을 해주고 변수명을 잘 지으면 문제가 없다고 한다. 어짜피 쓰는 것은 본인의 판단이고 위에 나열된 iterator나 람다 등 복잡한 형식에는 써주는게 좋을 것 같다.

위 글은 effective modern c++에 나오는 내용을 정리한 것입니다