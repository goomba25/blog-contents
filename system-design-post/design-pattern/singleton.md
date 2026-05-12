---
title: "[Design Pattern] Singleton Pattern (싱글톤 패턴)"
excerpt: "Singleton Pattern"
date: "2026-05-12"
categories: [system]
---

# Singleton Pattern ?

싱글톤 디자인 패턴은 쉽게 말해 "프로그램에서 객체를 오직 하나만 생성하도록 강제하는" 디자인 패턴입니다.

프로그램 전역에서 하나의 객체만 존재하도록 보장되며, 이는 공유 자원을 효율적으로 관리할 수 있다는 장점이 있습니다.

## Singleton 기본 형태

```cpp
class TestClass {
  public:
    static TestClass& getInstance() {
        static TestClass instance;
        return instance;
    }

  private:
    TestClass() = default;
    ~TestClass() = default;
    TestClass(const TestClass&) = delete;
    TestClass& operator=(const TestClass&) = delete;
    TestClass(TestClass&&) = delete;
    TestClass& operator=(TestClass&&) = delete;
};

int main() {
    auto& object = TestClass::getInstance();
}
```

## Singleton 구조 특징

### 유일한 객체 보장

Singleton의 객채가 프로그램에서 유일할 수 있는 이유는 아래와 같습니다.

1. private 생성자
2. 복사/대입 생성자 삭제
3. 이동/대입 생성자 삭제

이를 통해 개발자가 실수로 복사/이동/대입 하는 상황을 방지할 수 있습니다.

또한 static 변수를 사용하기 때문에, 최초로 `getInstance`를 호출하는 시점에 생성 및 초기화 되고,    
이후에 `getInstance`를 호출하더라도 생성된 static 변수를 반환하여 사용하게 됩니다.

### 복사/이동/대입 생성자 삭제

* 복사 생성자
* 복사 대입 생성자
* 이동 생성자
* 이동 대입 생성자

이렇게 총 4가지는 기본적으로 막아두고 시작하는 것을 권장한다고 합니다.

복사 생성자는 새 객체를 기존 객체로부터 복사하여 생성하기 때문에 "하나의 객체"를 보장하는 Singleton 디자인 패턴에 맞지 않습니다.   
복사 대입 생성자는 이미 존재하는 객체에 다른 객체의 값을 덮어쓰는데 사실 하나의 객체만 생성하는 Singleton에는 불필요할 수 있지만 원칙적으로 막는다고 합니다.

이동 생성자는 기존 객체의 자원을 새 객체로 옮기면서 객체를 생성하는데 "하나의 객체"를 보장하는 Singleton에서의 자원 이동은 위험하기 때문에 막아야 합니다.   
이동 대입 생성자 또한 자원 이동을 허용할 이유가 없기 때문에 막아야 합니다.

### 소멸자

Singleton 디자인 패턴은 따로 소멸자를 구현하지 않을 수 있다고 합니다.   
생성된 시점 이후로 static 객체는 계속 존재하고, 프로그램 종료 시 자동으로 소멸되기 때문입니다.

다만 의도를 명확하게 표현하기 위해 소멸자를 명시하는 것도 하나의 방법일 수 있습니다.

### 클래스 상속

Singleton 클래스는 기본적으로 상속하지 않는 방향으로 설계한다고 합니다.

객체가 하나만 존재하고, 생성 경로를 통제하고, 객체의 생명 주기도 제어하는데   
상속을 하면 이러한 특성이 쉽게 깨질 수 있기 때문입니다.

* 그렇다고 절대 상속을 하지 않는다는 건 아닙니다.