---
title:  "const vs readonly"
search: true
toc: true
toc_sticky: true
categories: 
  - dotnet
tag:
  - CSharp
  - const
  - Readonly
---

# const
- C# 에서 __상수__ 를 정의할 때 사용하는 키워드
- 변수만 Stack, Heap에 할당되므로 상수는 인스턴스가 존재하지 않는다.
- 컴파일 타임에 const로 선언된 상수를 사용한 코드를 모두 실제 상수 값으로 바꾼다.
```cs
...
public const int abc = 3;
...
Console.WriteLine(abc);
```
위의 코드는 컴파일 후
```cs
...
public const int abc = 3;
...
Console.WriteLine(3);
```
위와 같이 변경된다.

# readonly
- const와 다르게 런타임에서 정의된다.
- const보다 다양한 곳에서 활용된다. [https://docs.microsoft.com/ko-kr/dotnet/csharp/write-safe-efficient-code]
- 생성자를 통해서만 값을 수정할 수 있으며, 이후 새로운 참조가 할당되는 것을 막는 키워드기 때문에 const처럼 메모리 할당 개념으로 사용되지 않는다.

# 참고자료
- https://social.msdn.microsoft.com/Forums/vstudio/en-US/031580c1-d977-4013-bd86-552bd8b54edc/constants-memory-allocation?forum=csharpgeneral