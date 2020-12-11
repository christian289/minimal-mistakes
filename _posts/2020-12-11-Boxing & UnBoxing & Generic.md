---
title:  "Boxing & UnBoxing & Generic"
search: true
toc: true
toc_sticky: true
categories: 
  - dotnet
tag:
  - CSharp
  - Boxing & UnBoxing
  - Generic
  - Sync Block Index
  - Type Object Pointer
---

# 잘못된 개념
- 흔히들 Boxing은 'object로 형변환하는 작업'으로 착각하고 있다. 마찬가지로 UnBoxing 또한 'object에서 다른 타입으로 형변환 하는 작업'으로 착각하고 있다.
- 굳이 object가 아니더라도 아래와 같은 코드도 ValueType이 Reference Type으로 변환되므로 Boxing이다.

```cs
int a = 123;
string b = a.ToString();
```

```cs
ArrayList collection = new ArrayList();

for (int i = 99; i > 0; i--)
{
    collection.Add(i);
}
```

- 첫번째 예제코드는 object로 형변환 하지 않았어도 Boxing이다.
- System.ValueType은 Thread Stack에 할당될 수도 있고, Managed Heap에 할당될 수도 있는 타입이다. Thread Stack에만 할당된다는 편견은 버려라.

# .NET의 Boxing
- .NET의 포인터 변수 크기는 __x86에서 4Byte, x64에서 8Byte__ 다.
- .NET의 타입 객체 포인터(Type Object Pointer) 또는 메소드 테이블 포인터(Method Table Pointer) 변수의 크기는 __x86에서 4Byte, x64에서 8Byte__ 다.
- .NET의 동기화 블록 인덱스(Sync Block Index) 의 크기는 __x86에서 4Byte, x64에서 8Byte__ 다.
- Boxing이란, System.ValueType을 상속받는 struct 타입의 값이 Thread Stack 영역에 할당되고 이 값을 Managed Heap 영역으로 __복사(또는 새로 할당)__ 하는 것을 의미한다. __복사라는 것에 굉장히 많은 의미가 담겨있다.__ Stack영역에 값을 바로 할당하면 정말 값만 할당된다. 이 값을 Heap으로 복사할 때는 아래와 같은 정보가 추가적으로 필요하다.
  - Managed Heap을 참조할 수 있는 Thread Stack의 포인터변수
  - Managed Heap의 실제 인스턴스 공간
  - 타입 객체 포인터
  - 동기화 블록 인덱스
- 동기화 블록 인덱스는 Thread Stack의 포인터 변수가 참조하는 실제 주소의 시작부분에서 - 방향으로 4Byte, 8Byte 방향으로 생성된다.
- 위에서 설명한대로 Boxing이란 object로 타입캐스팅한다는 의미가 아닌, 위의 인스턴스를 __새로 할당한다는 개념__ 이다.
- 즉, stack에서 int형 변수를 사용하다가, object로 형변환을 하면, x86 기준으로 4Byte가 16Byte(Thread Stack 포인터 변수 4Byte + 타입 객체 포인터 4Byte + Managed Heap에 할당된 실제 인스턴스 크기 4Byte + 동기화 블록 인덱스 4Byte)가 되는 마법을 볼 수 있다.
- 또한 Boxing된 변수는 Thread Stack의 변수처럼 메소드가 종료되었다고 바로 사라지는 것이 아닌, Garbage Collector가 수집해야만 제거가 가능하므로 바로 제거되지도 않는다.
- Boxing을 하지말고 차라리 처음부터 Managed Heap에 할당해놓고 사용하는게 빠르고 효율적이며, Garbage가 덜 생성되는 방법이다.

# .NET의 UnBoxing
- UnBoxing은 Boxing보다 단순한 작업이다. 포인터 변수가 사라지고, 동기화 블록 인덱스가 사라지는 작업이므로 상대적으로 간편하다. 하지만 이 역시, Managed Heap에서 Thread Stack으로 __복사(또는 새로 할당)__ 하는 작업이므로 메모리 낭비가 맞다.

# .NET의 Generic
- 컬랙션 객체에 원래 ValueType의 값을 저장하고 싶다면 위의 예제처럼 ArrayList를 통해 Boxing이 되는 형태로 사용했지만, .NET Framework 2.0에서 Generic이 도입되면서 Boxing을 예방할 수 있게되었다.
- **Generic은 런타임 시점에서 인스턴스를 형성할 때 Generic Type으로 지정되어 인스턴스를 할당하기 때문에** Boxing이 아닌 처음부터 Reference type의 ValueType을 할당할 수 있다. 아래와 같은 구조라고 보면 된다.

```cs
class Sample
{
    int a { get; set; }
}
```

- 위의 int는 ValueType이지만 Class의 Property로 정의 되었으므로 Reference Type이다. Generic이 위와 같이 Reference Type의 ValueType을 사용할 수 있게한다.
- List<int>는 위와 같이 Reference Type으로 동작하므로 Boxing이 없고, 메소드 같은 경우 아래 코드를 보면 generic이 int형태로만 동작하기 때문에 boxing이 없다. 따라서 Boxing이 예방된다.

```cs
public T Sum<T>(T t1, T t2) where T : struct
{
    return t1 + t2;
}

var result = Sum<int>(3, 4);
Console.WriteLine(result); // 7
```

# Dictionary
- Dictionary는 Generic을 사용하면서 Key-Value 형태로 편하게 사용할 수 있는 클래스다. 객체지향 프로그래밍은 값에 대한 의미가 중요하므로, Key로 Enum을 사용한다면 명시적인 Key로서 구분이 용이할 것이다. 일반적인 Generic Class에서는 Enum을 사용해도 Boxing이 발생하지 않는다. 하지만 __Dictionary에서는 Key로 Enum 타입을 지정할 경우 특성상 Key Compare를 해야하기 때문에 내부적으로 Equals 메서드가 사용된다. 이 메서드는 object 형을 파라미터로 받고 있으므로 Enum이 object로 Boxing 된다.__ 따라서 Enum을 아래의 방법으로 Managed Heap 영역에 넣어두고 사용해야 Boxing이 발생하지 않는다.
- [https://www.slideshare.net/devcatpublications/enum-boxing-enum-ndc2019](https://www.slideshare.net/devcatpublications/enum-boxing-enum-ndc2019)
- [https://www.sysnet.pe.kr/2/0/11565](https://www.sysnet.pe.kr/2/0/11565)

# 참고자료
- [https://stackoverflow.com/questions/3815227/understanding-clr-object-size-between-32-bit-vs-64-bit](https://stackoverflow.com/questions/3815227/understanding-clr-object-size-between-32-bit-vs-64-bit)
- [https://docs.microsoft.com/en-us/archive/msdn-magazine/2005/may/net-framework-internals-how-the-clr-creates-runtime-objects#S7](https://docs.microsoft.com/en-us/archive/msdn-magazine/2005/may/net-framework-internals-how-the-clr-creates-runtime-objects#S7)
- [https://www.codingblocks.net/programming/boxing-and-unboxing-7-deadly-sins/](https://www.codingblocks.net/programming/boxing-and-unboxing-7-deadly-sins/)
- [https://docs.microsoft.com/ko-kr/dotnet/standard/generics/](https://docs.microsoft.com/ko-kr/dotnet/standard/generics/)