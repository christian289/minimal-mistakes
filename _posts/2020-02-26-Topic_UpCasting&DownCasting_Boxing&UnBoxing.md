---
title:  "[알쓸신잡 코딩]] UpCasting과 DownCasting // Boxing과 UnBoxing"
search: true
toc: true
toc_sticky: true
categories: 
  - C#
tag:
  - .NET
---

# Boxing 과 UnBoxing
- 닷넷 카톡방에서 **UpCasting**, **DownCasting** 이슈가 나왔었습니다.
객체지향 프로그래밍에서 상속받는 부모클래스가 있는 클래스가 부모 클래스로 형변환 할 시 **UpCasting**이라고 하고, 반대로 부모클래스를 자식 클래스로 형변환하는 것을 **DownCasting**이라고 합니다.
예를 들어,

```cs
interface IParent
{
  void Hello();
}

class ChildClass : IParent
{
  protected void Hello()
  {

  }
}

void Main()
{
  IParent a = new ChildClass();
}
```

위와 같은 코드는 ChildClass가 IParent 인터페이스를 상속받는데 IParent에 ChildClass형 인스턴스를 할당했으니 이것은 **UpCasting**이라고 볼 수 있겠습니다.
여기서 제가 **그럼 Boxing도 UpCasting의 종류입니까?**하고 한분께 여쭤봤습니다.
답은 No이며 완전히 다른 개념이라고 하셨지만 이해가 잘 되지 않았습니다.

제가 알고 있던 **Boxing**이란
객체지향에서 object Type이 모든 클래스의 시조이며, 여러가지 기본형이나, 사용자 정의 클래스나 object로 형변환하는 것을 Boxing, 반대로 형변환 하는 것을 UnBoxing 이라고 알았습니다.
즉,

```cs
object o1 = 123; // 이것은 Boxing
int i = (int)o1; // 이것은 UnBoxing

object o2 = new 사용자정의클래스(); // 이것은 Boxing??
사용자정의클래스 c1 = (사용자정의클래스)o2; // 이것도 UnBoxing??
```

으로 알았고, int형은 object형을 상속 받으니 UpCasting 인 줄 알았던 것입니다. 하지만 제가 틀렸습니다.
**Boxing&UnBoxing과 UpCasting&DownCasting은 완전히 다른 내용입니다.**

그 답은 [MSDN](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/boxing-and-unboxing)에 잘 나와있습니다.
MSDN을 보면, 가장 첫 문장에 Boxing에 대하여, **Boxing is the process of converting a value type to the type object or to any interface type implemented by this value type.** 라고 합니다.

C#에는 C에서 배웠던 것처럼, Value Type이 있고, Reference Type이 있습니다.
Value Type은 값 형식으로 struct로 정의되며, Reference Type은 참조 형식으로 class로 정의됩니다.
우리가 흔히 사용하는 예약어 변수인 **int는 사실 [Int32](https://docs.microsoft.com/ko-kr/dotnet/api/system.int32?view=netframework-4.8)라는 구조체**입니다.
C#의 자료형은 [MSDN](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/builtin-types/integral-numeric-types)에서 역시 자세히 확인하실 수 있습니다.

![CSharp_struct1]](/assets/images/UpCasting&DownCasting_Boxing&UnBoxing(1).PNG)

우리가 사실 예약어로 막연하게 사용중이던 기본 형들이 사실 모두 **구조체**였던 것입니다.
여기서 발생하는 의문은...**구조체는 클래스가 아니기 때문에 상속이 불가능**합니다.
따라서 object를 상속받을 수 없기 때문에 객체지향에서 원조 조상격 클래스인 object로 형변환이 될 수 없습니다.
그런데, 위에서 본 코드는 오류가 발생하지 않습니다.

```cs
object o1 = (object)123; // 이것은 Boxing
int i = (int)o1; // 이것은 UnBoxing
```

어떻게 된 것일까요? 여기서 다시 한 번 MSDN을 자세히 살펴볼 필요가 있습니다.

![CSharp_struct2]](/assets/images/UpCasting&DownCasting_Boxing&UnBoxing(2).PNG)

...? 구조체들은 상속이 불가능한데, 우리가 사용하는 int나 double 같은 기본형의 구조체들이 모두 ValueType이라는 클래스를 **상속**받고 있습니다.

다음은 C# 4.1.1, C# 4.1.3 언어 스펙에서 발췌한 문장입니다.

>4.1.1 The System.ValueType type
>All value types implicitly inherit from the class System.ValueType, which, in turn, inherits from class object. It is not possible for any type to derive from a value type, and value types are thus implicitly sealed (§10.1.1.2).

>4.1.3 Struct types
>A struct type is a value type that can declare constants, fields, methods, properties, indexers, operators, instance constructors, static constructors, and nested types.

설명을 보니 이 ValueType은 사용자가 직접 사용할 수는 없는 암시적 클래스이며, .NET의 **모든 값 형식이 이 ValueType을 상속**받는다고 합니다.
그러므로 우리가 사용하는 Int32나 Int64 등등의 값 형식은 ValueType을 받아 상속되어 정의된 구조체입니다.
사용자가 정의하는 구조체 역시 System.Object를 상속받는 System.ValueType을 **이미 상속 받고 있습니다.**

```cs
object o1 = (object)123; // 이것은 Boxing
int i = (int)o1; // 이것은 UnBoxing
```

따라서 위 코드로 Boxing을 설명하면,
Memory의 Stack 영역에 123이라는 Int32 struct 형식의 Value가 생성되며 이 123은 Stack이기 때문에 선언된 함수나 클래스의 스코프가 끝나면 사라집니다.
123이라는 Value는 object로 형변환 되기 위해 **추상클래스인 ValueType**으로 참조를 거쳐 object형으로 변환되고 123이라는 값이 동적할당이 됩니다.
위와 같은 **Stack 영역 메모리의 값을 Heap 영역 메모리로 할당해주는 것을 Boxing** 이라고 합니다.
반대로 **Heap 영역 메모리에서 Stack 영역 메모리로 할당되는 것을 UnBoxing**이라고 합니다.
따라서 사용자가 정의한 참조형 클래스의 객체가 object로 변환되는 것은 Boxing이라고 하지 않습니다. 굳이 말하면 UpCasting 이겠죠.

이상으로 MSDN 링크를 남기고 마치겠습니다.

object 정의: [System.Object](https://docs.microsoft.com/ko-kr/dotnet/api/system.object?view=netframework-4.8)
ValueType 정의: [System.ValueType](https://docs.microsoft.com/ko-kr/dotnet/api/system.valuetype?view=netframework-4.8)
Boxing & UnBoxing 정의: [Boxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/boxing-and-unboxing)

