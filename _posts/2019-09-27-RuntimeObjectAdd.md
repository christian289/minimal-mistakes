---
title:  "동적으로 객체 List에 추가하기"
search: true
categories: 
  - C#
tag:
  - C#
  - Reflection
  - ConstructorInfo
---

## Reflection 이란
> [리플렉션](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/reflection)은 어셈블리, 모듈 및 형식을 설명하는 개체(Type 형식)를 제공합니다. 리플렉션을 사용하면 동적으로 형식 인스턴스를 만들거나, 형식을 기존 개체에 바인딩하거나, 기존 개체에서 형식을 가져와 해당 메서드를 호출하거나, 필드 및 속성에 액세스할 수 있습니다. 코드에서 특성을 사용하는 경우 리플렉션은 특성에 대한 액세스를 제공합니다. -MSDN에서 발췌

위에서 설명한 것처럼 Reflection은 코드를 이미 실행한 Runtime에서 동적으로 객체를 핸들링할 수 있는 기능을 제공합니다.

여러 사용방식이 있겠지만 이번 글에서는 **ConstructorInfo** 를 사용합니다.

전체 코드를 먼저 보겠습니다.

```cs
public List<Member> Regist()
{
    var MemberListType = from Member in Assembly.GetExecutingAssembly().GetTypes()
                            where Member.IsClass && Member.IsSubclassOf(typeof(Member))
                            select Member;

    List<Member> MemberList = new List<Member>();

    foreach (Type MemberType in MemberListType)
    {
        Type[] emptyType = Type.EmptyTypes;
        ConstructorInfo emptyConstructor = MemberType.GetConstructor(emptyType);
        MemberList.Add((Member)emptyConstructor.Invoke(new object[] { }));
    }

    return MemberList;
}
```

코드 설명하겠습니다.
```cs
var MemberListType = from Member in Assembly.GetExecutingAssembly().GetTypes()
                            where Member.IsClass && Member.IsSubclassOf(typeof(Member))
                            select Member;
```
1. 현재 어셈블리(프로젝트 결과물)의 안에 있는 모든 Type을 가져옵니다.
2. 그 중에서 **클래스 타입(IsClass))**이면서 **Member라는 클래스를 상속받는 타입(IsSubclassOf)**을 선택합니다.
3. MemberListType에 저장합니다.

```cs
foreach (Type MemberType in MemberListType)
{
    Type[] emptyType = Type.EmptyTypes;
    ConstructorInfo emptyConstructor = MemberType.GetConstructor(emptyType);
    MemberList.Add((Member)emptyConstructor.Invoke(new object[] { }));
}
```
1. foreach를 통해 MemberListType에서 하나씩 가져옵니다.
2. 파라미터가 없는 생성자 정보(ConstructorInfo)를 가져옵니다.
3. 생성자 정보를 생성할 때 파라미터가 없는 정보를 가져왔기 때문에 new object 배열을 사용해서 빈 값으로 넣습니다.
4. Invoke함수로 생성자를 만들고 Member로 자료형을 변환하여 리턴객체에 추가해줍니다.

C#에서 Reflection을 사용할 때 주의하실 점은,
# 빌드 후 Runtime 환경에서 객체정보를 가져오는 것이기 때문에 컴파일 단계에서 IDE가 버그를 잡아줄 수 없습니다.
그렇기 때문에 배포 전에 개발자가 예상한대로 동작하는지 확실한 테스트가 강제요구됩니다.