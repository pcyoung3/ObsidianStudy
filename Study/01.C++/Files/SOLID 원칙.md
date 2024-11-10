---
Index: "[[1. CppIndex]]"
tags:
  - Study
  - cpp
---
   
> [!question] SOLID 원칙이란?
> 객체지향에서 개발과 유지보수를 쉽게 할 수 있도록 하는 가이드 라인
   
   
## 1. SRP(Single Responsibility Principle) : 단일 책임 원칙
---
> [!info] 클래스는 오직 하나에 대해서만 책임져야 한다

![[Pasted image 20230607092732.png]]
클래스가 하는 일이 하나여야 하고 여러가지면 안된다.
클래스가 하는 일이 한개라면 해당 클래스가 변경되는 이유도 한가지여야 한다.
   
### 1-1. 나쁜예시
```cpp
class Worker
{
	void Cook();
	void GardenWork();
	void Paint();
	void Drive();
};
```
   
### 1-2. 좋은예시
```cpp
class Chef
{
	void Cook();
};

class Gardener
{
	void GardenWork();
};

class Painter
{
	void Paint();
};

class Driver
{
	void Drive();
};
```
   
   
## 2. OCP(Open-Closed Principle) : 개방 폐쇄 원칙
---
> [!info] 클래스에 기능을 수정 또는 추가 할 때 코드를 추가하는 방식으로 진행. 기존 코드를 수정하는 방식으로 진행하면 안됨

![[Pasted image 20230607093501.png]]
클래스의 현재 코드를 변경하는 것은 해당 클래스를 사용하는 모든 시스템에 영향을 준다.
따라서 어떤 기능을 추가할 때 기존 코드를 수정하는 것보다 새로운 코드를 추가하는 방향이 더 좋다
   
### 나쁜예시
```cpp
class Person
{
	void Todo()		//놀기를 추가할 경우 기존 사용되던 Todo함수를 수정해야됨
	{
		cout << "밥먹기" << endl;
		cout << "공부하기" << endl;
		cout << "운동하기" << endl;
		cout << "잠자기" << endl;
	}
};
```
   
### 좋은예시
```cpp
class Person2	//놀기를 추가할 경우 함수를 하나 추가하면 됨
{
	void Eating();
	void Study();
	void Exercise();
	void Sleep();
};
```
   
   
## 3. LSP(Liskov Substitution Principle) : 리스코프 치환 원칙
---
> [!info] 자식은 부모의 모든 기능에 관해서 대체가 가능해야 한다

![[Pasted image 20230607094426.png]]
자식클래스는 부모클래스 기능을 똑같이 수행할 수 있어야 함
똑같은 요청에 똑같은 응답을 할 수 있어야 하고 응답의 타입도 같아야 함
   
### 3-1. 나쁜예시
```cpp
class CoffeeMaker
{
public:
	void MakeCoffee()
	{
		cout << "Coffee" << endl;
	}
};

class WaterMaker : public CoffeeMaker
{
public:
	void MakeCoffee()
	{
		cout << "Water" << endl;
	}
};
```
   
### 3-2. 좋은예시
```cpp
class CoffeeMaker
{
public:
	void MakeCoffee()
	{
		cout << "Coffee" << endl;
	}
};

class CappuccinoMaker : public CoffeeMaker
{
public:
	void MakeCoffee()	//오버라이딩
	{
		cout << "Cappuccino" << endl;
	}
};

class latteMaker : public CoffeeMaker
{
public:
	void MakeLatte()	//기존함수 냅두고 다른 함수 추가
	{
		cout << "Latte" << endl;
	}
};
```
   
   
## 4. ISP(Interface Segregation Principle) : 인터페이스 분리 원칙
---
> [!info] 클라이언트는 사용하지 않는 메서드에 의존적이면 안됨

![[Pasted image 20230607095812.png]]
클래스는 필요한 기능만 가지고 있어야 한다. 다른 기능들은 삭제하거나 다른 곳으로 이동 시켜야 한다.

   
### 4-1. 나쁜예시
```cpp
class Robot
{
	void Spin();
	void RotateArms();
	void WiggleAntennas();
};
```
   
### 4-2. 좋은예시
```cpp
class ISpinWheel
{
public:
	virtual void Spin() = 0;
};

class IArms
{
public:
	virtual void RotateArms() = 0;
};

class IAntennas
{
public:
	virtual void WiggleAntennas() = 0;
};

class RobotA : public ISpinWheel, public IArms, public IAntennas
{
	void Spin();
	void RotateArms();
};

class RobotB : public ISpinWheel, public IArms
{
	void Spin();
	void RotateArms();
};
```
   
   
## 5. DIP(Dependency Inversion) : 의존성 역전 원칙
---
> [!info] 추상은 구체에 의존하지 않아야 하며, 구체는 추상에 의존적이여야 함. 상위모듈은 하위모듈에 의존해서는 안됨

![[Pasted image 20230607102100.png]]
말이 가장 어려운데 상위모듈(고수준) 은 인터페이스, 추상객체를 의미하고 하위모듈(저수준)은 사용자가 사용하게 되는 일반 클래스들을 의미한다.
또한 추상(인터페이스), 구현(클래스)의 측면으로 봐도 같다.
즉, 일반 클래스들이 인터페이스나 추상클래스에 의존되어야 한다는 말이다.
   
클래스간의 의존성을 최소한으로 해야 된다는 말
어떤 방식으로 의존성을 최소화 시킬것인가? -> Iterator 패턴!
   
### 5-1. 나쁜예시
```cpp
class cat {};
class dog {};
class bird {};

class zoo	//zoo에 동물을 추가할 때마다 맴버변수를 늘려줘야 함
{
	cat c;
	dog d;
	bird b;
};
```
   
### 5-2. 좋은예시
```cpp
class IAnimal
{
	virtual void Move() = 0;
};

class cat : public IAnimal
{
	virtual void Move() override {}
};

class dog : public IAnimal
{
	virtual void Move() override {}
};

class bird : public IAnimal
{
	virtual void Move() override {}
};

class zoo
{
	zoo()
	{
		//동물의 추가가 얼마든지 자유로움
		Animals.emplace_back(new cat());
		Animals.emplace_back(new dog());
		Animals.emplace_back(new bird());
	}
	list<IAnimal*> Animals;
};
```