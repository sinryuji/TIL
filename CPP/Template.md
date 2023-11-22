# Template

## 템플릿(Template)이란?

함수를 어떤 자료형에서든 동작하도록 구현하려면 오버로딩을 통해 구현을 하면 되긴 하지만 사실 이것도 자료형마다 함수를 따로 구현한다는게 매우 귀찮다.

그래서 등장한 것이 아닌가 하는 것이 Template이라는 놈이다.

템플릿을 사용하면 함수나 클래스는 개별적으로 재구현하지 않아도 여러 자료형에서 사용할 수 있다.

`함수 템플릿(Function Template)`과 `클래스 템플릿(Class Template)`로 나뉜다,

## 함수 템플릿(Function Template)

- 함수를 만들 때 기능은 명확하지만 자료형을 **모호하게 두는 것.**

만약 두 수를 더하는 sum이라는 함수를 만들 건데 int와 double에서 동작하도록 구현하고 싶다.

그럼 int 자료형을 사용하는 놈과 double 자료형을 사용하는 놈 두 가지를 오버로딩으로 구현해야 한다. 다음과 같이.

```cpp
int sum(int a, int b)
{
	return a + b;
}

double sum(double a, double b)
{
	return a + b;
}
```

하지만 함수 템플릿을 사용하면 위와 같이 자료형을 제외하곤 모든게 똑같은 함수를 두 개나 만드는 거지같은 짓을 하지 않아도 된다.

```cpp
template <typename T>
T sum(T a, T b)
{
    return a + b;
}
```

인자를 두개 받는데 서로의 타입이 달라도 가능하다.

```cpp
template <class T1, class T2>
void printAll(T1 a, T2 b)
{
	std::cout << a << std::endl;
	std::cout << b << std::endl;
}
```

템플릿으로 구현한 sum 함수는 다음과 같이 사용한다.

```cpp
int	main(void)
{
	int a = 1, b = 2;	
	double c = 1.3, d = 2.6;

	std::cout << sum<int>(a, b) << std::endl;
	//std::cout << sum(a, b) << std::endl;
	std::cout << sum<double>(c, d) << std::endl;
}
```

주석과 같이 <int>와 같이 자료형을 표시해주지 않고 사용해도 된다. 그럴 경우 a와 b가 int이므로 알아서 int 타입 + 연산을 수행한다. 만약 서로 다른 자료형을 주면?

```cpp
std::cout << sum<int>(a, c) << std::endl;
std::cout << sum(a, c) << std::endl;
```

위의 케이스에서 첫째줄은 double인 c가 int로 캐스팅되어 함수가 동작한다. 반면 두번째 줄의 경우는 컴파일 타임에러 에러가 발생된다.

### 함수 템플릿 특수화(Function Template Specialization)

템플릿을 사용할때 특정 자료형에 대해서는 다른 동작을 구현하고 싶을 때 사용한다.

char 포인터는 + 연산을 할 수 없으므로 char 포인터에 한해서 다르게 동작을 구현하고 싶다면 다음과 같이 구현하면 된다.

```cpp
template <>
char* sum(char* a, char* b)
{
	char* str = "char pointer does'nt + calculate";
	return str;
}
```

## 클래스 템플릿(Class Template)

- 클래스 내부 멤버 변수에 템플릿을 적용 할 수 있다.
- 멤버 함수를 외부에서 구현 시 template 선언을 다시 해줘야 한다.
- 클래스 템플릿을 객체를 생성할때 타입을 정해준다.

```cpp
template <class T>
class Person
{
	private:
		std::string name;
		T height;
		
	public:
		Person(std::string name, T height) : name(name), height(height) {}
		void	print(void)
		{
			std::cout << "name : " << this->name << std::endl;
			std::cout << "hegiht : " << this->height << std::endl;
		}
		void	setName(std::string name);
		void	setHeight(T hegiht);
};

template <typename T>
void	Person<T>::setName(std::string name)
{
	this->name = name;
}

template <typename T>
void	Person<T>::setHeight(T hegiht)
{
	this->height = hegiht;
}

int	main(void)
{
	Person<int> p1("kim", 180); // 객체를 생성할 때 <int>를 통해 타입을 정해준다.
	Person<std::string> p2("kim", "180cm");
	p1.print();
	p2.print();
	p1.setName("park");
	p1.setHeight(190);
	p2.setName("park");
	p2.setHeight("190cm");
	p1.print();
	p2.print();
}
```

### 클래스 템플릿의 특징

1. 하나 이상의 템플릿 인수를 가지는 클래스를 선언할 수 있다.
    
    ```cpp
    template <typename T, int i> // 두 개의 템플릿 인수를 가지는 클래스 템플릿을 선언함.
    class X
    { ... }
    ```
    
2. 디폴트 템플릿 인수를 선언할 수 있다.
    
    ```cpp
    template <typename T = int, int i> // 디폴트 템플릿 인수의 타입을 int형으로 명시함.
    class X
    { ... }
    ```
    
3. 클래스 템플릿을 상속 받을 수 있다.
    
    ```cpp
    template <typename Type>
    class Y : public X <Type> // 클래스 템플릿 X를 상속받음.
    { ... }
    ```
    

### 클래스 템플릿 특수화(Class Template Specialization)

함수 템플릿과 마찬가지로 클래스 템플릿도 특수화가 가능하다.

double 타입에 대한 특수화는 다음과 같다.

```cpp
template<> class X<double> { ... };
```

### 부분 특수화(Partical Specialization)

만약 템플릿 인수가 두개 이상인데 그 중 하나에만 특수화를 진행하고 싶으면 `부분 특수화(Partical Specialization)`을 사용할 수 있다.

```cpp
// 일반 클래스 템플릿 특수화
template <typename T>
class X {};

template <>
class X<double> {};

// 클래스 템플릿 부분 특수화
template <typename T1, typename T2>
class Y {};

template <typename T1>
class Y<T1, double> {};

// 여러개 템플릿 인수를 모두 특수화
template <typename T1, typename T2>
class Z {};

template <>
class Z<double, double> {};
```

## \<typename T\> vs \<class T\>

위의 코드에서 클래스에 템플릿을 사용 할 때는 `<class T>`를 사용한 것을 확인 할 수 있다.

이 두개는 기능상 서로 완전히 동일하며 그냥 이름만 다르다고 봐도 된다.

c++17 이전까지는

```cpp
template < template < typename, typename > class Container, typename Type >
class Example
{
     Container< Type, std::allocator < Type > > baz;
};
```

다음과 같은 케이스에서 class 키워드를 사용해야 한다고 하지만 위와 같은 template 중첩문을 사용할 일이 잘 있지도 않을 뿐더러 c++17 이후로는 typename을 사용해도 된다고 하니 이제 완전히 동일한 키워드로 봐도 괜찮을 것 같다.

추가로 class는 c++ 표준화 이전에 사용하던 키워드라고 한다. c++가 표준화 되면서 더 명확한 이름인 typename으로 바뀌었고 혼돈을 줄이기 위해 현재 두 키워드 모두 지원한다고 보면 될 것 같다.

## 중첩 클래스 템플릿(Nested Class Template)

c++에서는 클래스나 클래스 템플릿 내에 또다른 템플릿을 중첩하여 정의할 수 있으며, 이러한 템플릿을 멤버 템플릿(Member Template)라고 부른다.

멤버 템플릿 중에서도 클래스 템플릿을 중첩 클래스 템플릿(Nested Class Template)이라 부른다.

이러한 중첩 클래스 템플릿은 바깥 클래스의 범위 내에서 클래스 템플릿으로 선언되며, 정의는 바깥 클래스의 범위 내에서 뿐만 아니라 밖에서도 가능하다.

```cpp
template <typename T>
class X
{
    template <typename U>
    class Y
    {
        ...
    }
    ...
    }

int main(void)
{
    ...
}

template <typename T>
template <typename U>
X<T>::Y<U>::멤버함수이름()
{
    ...
}
```

바깥 클래스 X에 중첩 클래스 템플릿 Y를 정의하면 `클래스 탬플릿의 탬플릿 인수(template <typename T>)`와 `멤버 템플릿의 템플릿 인수(template <typename U>)`가 모두 명시되어야 한다.

# 참고

[[C++] template(템플릿)에 관하여 2 (클래스 템플릿, 템플릿 특수화)](https://blockdmask.tistory.com/45)

[Difference of keywords 'typename' and 'class' in templates?](https://stackoverflow.com/questions/2023977/difference-of-keywords-typename-and-class-in-templates)

[[C++] template에서 typename과 class의 차이는?](https://blog.naver.com/PostView.nhn?blogId=oh-mms&logNo=222030206308)

[코딩교육 티씨피스쿨](http://www.tcpschool.com/cpp/cpp_template_class)
