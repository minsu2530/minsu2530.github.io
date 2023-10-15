---
title: SOLID
layout: default
parent: Design Pattern
grand_parent: Programming
---
# SOLID
GO를 가지고 [SOLID]를 설명한 글의 번역본입니다.

{: .no_toc }
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## What is SOLID?

소프트웨어는 복잡하고 그 복잡도 안에서 지속적으로 발전해야합니다. 문제 없이 지속적으로 발전하기 위해서는 **"high-quality"**, **"mainatainable"**, **"scalable"**한 코드를 작성할 수 있도록 해주는 디자인 원칙을 가지고 있는것이 중요합니다. 이런 원칙중 하나가 **SOLID**입니다.

(제가 읽었던 블로그 포스팅 중 하나에서는[^1], 이것을 객체지향 설계의 기초자세라고 생각한다고 합니다. 모두가 알고 있지만 완벽하게 하기는 어려운..)

SOLID는 각 원칙의 앞글자를 따서 만든 단어입니다.
- S : Single Responsibility Principle (SRP)

이번 블로그 포스트에서는, SOLID의 각 원칙들의 의미에 대해 자세히 살펴보고, 그것들이 어떻게 적용될 수 있는지 GO언어로 알아볼것입니다. SOLID design pattern에 대해 이해하는것은 high-quality code를 작성하는데 있어 매우 중요한 부분입니다. 이제부터 하나씩 알아보도록 하겠습니다.

## S - Single Responsibility Principle (SRP)

> 하나의 struct는 반드시 하나의 책임만을 가져야한다(하나의 struct의 바뀌어야할 이유는 하나뿐이어야한다.)

이 원칙을 사용했을 때 한 struct에 대한 변경이 한 부분에서만 발생하면 되기 때문에, 코드를 clean하고 mainatainable하게 할 수 있습니다. 



### S - Example in Golang
우리에게 `Employee`라는 struct가 있다고 해봅시다. 이 struct는 employee의 name, salary, address를 관리합니다.

``` golang
type Employee struct {
    Name    string
    Salary  float64
    Address string
}
```

SRP에 따르면, 하나의 Struct에는 한개의 Responsibility만을 가져야합니다. 따라서 이런 케이스에서는 Employee를 두개의 분리된 struct로 쪼개는것이 나을 수 있습니다. 즉, `Employee`를 `EmployeeInfo`와 `EmployeeAddress`로 쪼개는것이죠.

```golang
type EmployeeInfo struct {
    Name   string
    Salary float64
}

type EmployeeAddress struct {
    Address string
}
```
이제 각각의 struct에 대해 서로 다른 책임을 관리하는 분리된 함수를 가질 수 있게 되었습니다.
```golang
func (e EmployeeInfo) GetSalary() float64 {
    return e.Salary
}

func (e EmployeeAddress) GetAddress() string {
    return e.Address
}
```
SRP를 따름으로써, 각각의 struct는 명확하고 specific한 책임을 가지게 되었습니다. 이를 통해 우리의 코드는 더 maintainable하고 이해하기 쉬워졌습니다. 달리 말해, salary를 변경해야한다거나 address를 handling해야하는 상황이 되었을 때, 여러 코드들을 뒤지는게 아니라 어디를 봐야할지 정확하게 알 수 있게 되었습니다.

## O - Open-Closed Principle (OCP)

> 하나의 struct는 확장에는 Open되어있어야하지만, 변경에는 Closed되어 있어야한다.(struct의 행동은 코드를 변경하지 않고 확장가능해야한다.)

이런 원칙은 변화하는 requirement속에서 코드를 flexible하고 adaptable하게 유지할 수 있도록 해줍니다.


### O - Example in Golang
payment system을 하나 만든다고 해봅시다. 이 시스템은 신용카드 결제 처리가 가능해야합니다. 그리고 나중에는 다른 결제 방식도 가능할 수 있도록 충분히 flexible해야합니다.

```golang
package main

import "fmt"

type PaymentMethod interface {
  Pay()
}

type Payment struct{}

func (p Payment) Process(pm PaymentMethod) {
  pm.Pay()
}

type CreditCard struct {
  amount float64
}

func (cc CreditCard) Pay() {
  fmt.Printf("Paid %.2f using CreditCard", cc.amount)
}

func main() {
  p := Payment{}
  cc := CreditCard{12.23}
  p.Process(cc)
}
```
OCP에 따르면, 우리의 `Payment` struct는 extension에는 열려있고 modification에는 닫혀있습니다. `PaymentMethod` interface를 사용중이기 때문에, 새로운 payment method를 추가한다고 해서 `Payment`의 동작을 바꿀 필요는 없지만, 다음과 같이 `Paypal`과 같은 payment method를 추가할 수 있기 떄문입니다.
```golang
type PayPal struct {
  amount float64
}

func (pp PayPal) Pay() {
  fmt.Printf("Paid %.2f using PayPal", pp.amount)
}

// then in main()
pp := PayPal{22.33}
p.Process(pp)
```

## L - Liskov Substitution Principle (LSP)

> 서브타입은 언제나 base 타입으로 교체할 수 있어야한다.

적용방법은 아래와 같습니다.[^1]
1. 만약 두 객체가 똑같은 일을 한다면, 둘을 하나의 클래스로 표현하고 이들을 구분할 수 있는 필드를 둡니다.
2. 똑같은 연산을 제공하지만, 살짝씩 다른 연산이 필요하다면, 공통의 인터페이스를 만들고 둘이 이들을 구현하도록 합니다.
3. 공통된 연산이 없다면, 완전 별개의 2개의 클래스를 만듭니다.
4. 만약 두 개체가 하는 일에 추가적으로 무언가를 더 한다면, 구현 상속을 이용한다.

### L - Example in Golang
`Animal`이라는 struct를 한번 생각해봅시다.
```golang
type Animal struct {
  Name string
}

func (a Animal) MakeSound() {
  fmt.Println("Animal sound")
}
```
이제 특정 `Animal`의 타입을 나타내는 `Bird`라는 struct를 만들어봅시다.
```golang

type Bird struct {
  Animal
}

func (b Bird) MakeSound() {
  fmt.Println("Chirp chirp")
}

type AnimalBehavior interface {
  MakeSound()
}

// MakeSound represent a program that works with animals and is expected
// to work with base class (Animal) or any subclass (Bird in this case)
func MakeSound(ab AnimalBehavior) {
  ab.MakeSound()
}

a := Animal{}
b := Bird{}
MakeSound(a)
MakeSound(b)
```
이렇게 구현하였을 때 우리는 `Animal`이라는 superclass가 `Bird`라는 subclass로 replacable(코드의 흐름상 전혀 영향을 주지 않음)하게 됩니다. 달리 말해, Bird라는 subtype의 오브젝트는 어디에서는 Animal이라는 base type으로서 사용이 가능하게 되었습니다.

## I - Interface Segregation Principle (ISP)
> client는 자신이 사용하지 않는 인터페이스는 구현하지 않아야한다.(하나의 일반적인 인터페이스보다는 여러개의 구체적인 인터페이스가 낫다.)

이는 인터페이스는 가능한한 작고 specific하게 디자인되어야만 한다는 것을 의미합니다.

## D - Dependency Inversion Principle (DIP)
> high level module은 low level module에 의존해서는 안되며, 두 모듈 모두 추상화에 의존해야만 한다.

이는 컴포넌트간 coupling을 줄이기 위함이며, 코드를 flexible하고 maintainable하게 만들어줍니다.

### D - Example in Golang
`Worker`와 `Supervisor` struct가 있다고 해봅시다. `Worker`는 company의 worker를 의미하고, `Supervisor`는 company의 supervisor를 의미합니다. 그렇다면 아래와 같이 같은 id와 name을 가지는 class로 구현이 가능합니다.

```golang
type Worker struct {
  ID int
  Name string
}

func (w Worker) GetID() int {
  return w.ID
}

func (w Worker) GetName() string {
  return w.Name
}

type Supervisor struct {
  ID int
  Name string
}

func (s Supervisor) GetID() int {
  return s.ID
}

func (s Supervisor) GetName() string {
  return s.Name
}
```
둘의 highlevel module로서 `Department`를 정의합시다. 이 struct는 low level module인 worker와 supervisor의 정보를 저장하고 있을 필요가 있고 아래와 같이 구현이 가능합니다. 이는 DIP관점에서 anti pattern입니다.
```golang
type Department struct {
  Workers []Worker
  Supervisors []Supervisor
}
```
DIP에 따르면, high-level module은 low-level module에 depend해서는 안됩니다. 대신에 둘 다 abstraction에 depend해야합니다. 위의 anti-pattern example을 수정하기 위해 우리는 worker와 supervisor의 추상화인 Employee를 구현한 후 Department가 Employee에 의존하게 만들 수 있습니다.
```golang
type Employee interface {
  GetID() int
  GetName() string
}

type Department struct {
  Employees []Employee
}
```

이를 통해 Department는 세부적인 구현인 `Worker`, `Supervisor`에 의존하는 것이 아닌, abstraction인 `Employee`에 의존할 수 있도록 만들 수 있다. 또한 이를 통해 worker또는 supervisor의 코드의 변경이 `Department`에 영향이 가지 않게 되었습니다.

---
## 마치며

SOLID에 대해 간단한 예시가 있는 글을 봐서 번역해 보았습니다. 번역을 다 하고 보니 간단한 예시와 함께 SOLID에 대한 Introduction과 같다는 느낌을 받았습니다. 추가적으로 아래의 링크들을 참고하여 작성하였습니다.

- [https://www.nextree.co.kr/p6960/](https://www.nextree.co.kr/p6960/)
- [https://stackoverflow.blog/2021/11/01/why-solid-principles-are-still-the-foundation-for-modern-software-architecture/](https://stackoverflow.blog/2021/11/01/why-solid-principles-are-still-the-foundation-for-modern-software-architecture/)




---
[^1]: [https://www.nextree.co.kr/p6960/]


[https://www.nextree.co.kr/p6960/]: https://www.nextree.co.kr/p6960/
[SOLID]: https://hackernoon.com/go-design-patterns-an-introduction-to-solid