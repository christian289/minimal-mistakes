---
title:  "Intern Pool (Literal String Pool)"
search: true
toc: true
toc_sticky: true
categories: 
  - dotnet
tag:
  - CSharp
  - String
  - Intern Pool
---

.NET Runtime에서는 컴파일하고 빌드를 하면 많은 정보들이 생성된다. 특히 문자열은 메모리 관리차원에서 Intern Pool이라는 곳에 등록되는데, 이곳에 등록되는 문자열은 '리터럴 문자열'만 저장된다.

리터럴 문자열이란, 직접 소스 코드에서 __"Hello World" 처럼 쌍따옴표로 묶어서 코딩한 문자열__ 들이다. 이 문자열들만 골라서 Runtime시 Intern Pool에 등록해놓고, 문자열 객체를 여러개생성하지 않도록 한다. 

```cs
string s1 = "Hello";
string s2 = "Hello";
string s3 = "Hello";
string s4 = "Hello";
string s5 = "Hello";
```
위와 같이 코딩하면 Hello 객체는 런타임에서 5개가 생성되지 않고 모두 참조하는 주소가 Intern Pool을 가리킨다. ReferenceEqual 메서드를 통해 주소 비교를 할 수 있으며, 실행하면 모두 참조 주소가 같다고 나온다.

하지만 런타임에서 생성해낸 똑같은 문자열일 경우 리터럴 문자열 취급을 하지 않고 새로운 객체를 만든 것이기에 다른 주소를 참조한다. 아래의 코드를 보자.

```cs
string s1 = "Hello World";
string s2 = string.Concat("Hello", " World");
```
이 경우 ReferenceEquals(s1, s2)는 false 결과이다. s1의 경우 코딩을 쌍따옴표로 묶은 리터럴 문자열 취급이며, s2 문자열은 새로 문자열을 조합해서 새로운 객체를 리턴했으므로 리터럴 문자열이 아닌 일반적인 런타임 문자열 객체가 된다. 따라서 이 경우에는 Hello World 문자열 객체가 2개가 생긴다.

또한 Intern Pool에 등록되는 리터럴 문자열은 Managed Heap에 할당되지 않으며, 그렇기 때문에 당연히 Garbage Collector의 대상이 아니다. 따라서 프로그램이 끝나는 순간까지 해제되지 않는다.

리터럴 문자열은 아니지만 자주 사용될 것 같은 문자열을 Intern Pool에 등록하는 방법이 있는데, [string.Intern](https://docs.microsoft.com/ko-kr/dotnet/api/system.string.intern?view=net-5.0) 메서드를 사용하는 것이다. 자주 사용하는 문자열 객체를 Intern Pool 에 등록하고 사용한다면 임시 문자열 객체가 지속적으로 생성되는 일은 없을 것이며 따라서 0세대 가비지 컬렉션이 그만큼 할 일이 줄게 될 것이다. 그렇다고 Intern Pool에 문자열을 엄청나게 등록해놓고 사용하는 행동은 안하는 것이 좋은 것이, 언급한대로 Garbage Collection의 대상이 아니기 때문에 쓸데없이 메모리를 차지하는 것이고, 메모리를 차지하고 있는만큼 Garbage Collector가 메모리 부족현상을 빨리 느껴, Garbage Collection을 그만큼 더 빈번하게 할 것이다. 그럼 Garbage Collection Thread가 동작하는 동안 모든 Thread가 일시 정지하게 되고, 이는 곧 성능저하다.

추가적으로, 궁금해서 더 찾아봤는데
.NET 에는 문자열을 연결할 때 string.Format 메서드가 존재하고, C# 6.0에서는 string Interpolation(문자열 보간) 기능을 통해 string.Format을 좀 더 직관적으로 사용할 수 있게되었다. (string.Format과 문자열 보간기능은 완전히 동일하며, 문자열 보간으로 작성된 코드는 컴파일 타임에 string.Format으로 치환되었다가 MSIL로 컴파일 된다.)
string.Format에선 쌍따옴표로 묶인 문자열 안에 .NET에서 지원하는 문자열 포멧이 들어 있는데, 그럼 이 문자열도 Intern Pool과 관계가 있을까?

__답은 아니다.__

.NET Framework 4.8 Reference Source를 찾아보면 string.Format은 내부적으로 StringBuilder를 사용하고 있으며, 포멧 문자열로 작성된 문자열을 StringBuilder로 작성하여 문자를 하나하나 추가하고 파싱하여 포멧문자열을 만나면 해당 포멧형식으로 바꿔주고 return 할 때 최종적으로 GetStringAndRelease메서드에서 StringBuilder.ToString()을 호출한다.

```cs
public static String Format(String format, params Object[] args) {
    if (args == null)
    {
        // To preserve the original exception behavior, throw an exception about format if both
        // args and format are null. The actual null check for format is in FormatHelper.
        throw new ArgumentNullException((format == null) ? "format" : "args");
    }
    Contract.Ensures(Contract.Result<String>() != null);
    Contract.EndContractBlock();
            
    return FormatHelper(null, format, new ParamsArray(args));
}

private static String FormatHelper(IFormatProvider provider, String format, ParamsArray args) {
    if (format == null)
        throw new ArgumentNullException("format");
            
        return StringBuilderCache.GetStringAndRelease(
            StringBuilderCache.Acquire(format.Length + args.Length * 8).AppendFormatHelper(provider, format, args));
}

public static string GetStringAndRelease(StringBuilder sb)
{
    string result = sb.ToString();
    Release(sb);
    return result;
}

public static StringBuilder Acquire(int capacity = StringBuilder.DefaultCapacity)
{
    if(capacity <= MAX_BUILDER_SIZE)
    {
        StringBuilder sb = StringBuilderCache.CachedInstance;
        if (sb != null)
        {
            // Avoid stringbuilder block fragmentation by getting a new StringBuilder
            // when the requested size is larger than the current capacity
            if(capacity <= sb.Capacity)
            {
                StringBuilderCache.CachedInstance = null;
                sb.Clear();
                return sb;
            }
        }
    }
    return new StringBuilder(capacity);
}

internal StringBuilder AppendFormatHelper(IFormatProvider provider, String format, ParamsArray args) {
    if (format == null) {
        throw new ArgumentNullException("format");
    }
    Contract.Ensures(Contract.Result<StringBuilder>() != null);
    Contract.EndContractBlock();
 
    int pos = 0;
    int len = format.Length;
    char ch = '\x0';
 
    ICustomFormatter cf = null;
    if (provider != null) {
        cf = (ICustomFormatter)provider.GetFormat(typeof(ICustomFormatter));
    }
 
    while (true) {
        int p = pos;
        int i = pos;
        while (pos < len) {
            ch = format[pos];
 
            pos++;
            if (ch == '}')
            {
                if (pos < len && format[pos] == '}') // Treat as escape character for }}
                    pos++;
                else
                    FormatError();
            }
 
            if (ch == '{')
            {
                if (pos < len && format[pos] == '{') // Treat as escape character for {{
                    pos++;
                else
                {
                    pos--;
                    break;
                }
            }
 
            Append(ch);
        }
 
        if (pos == len) break;
        pos++;
        if (pos == len || (ch = format[pos]) < '0' || ch > '9') FormatError();
        int index = 0;
        do {
            index = index * 10 + ch - '0';
            pos++;
            if (pos == len) FormatError();
            ch = format[pos];
        } while (ch >= '0' && ch <= '9' && index < 1000000);
        if (index >= args.Length) throw new FormatException(Environment.GetResourceString("Format_IndexOutOfRange"));
        while (pos < len && (ch = format[pos]) == ' ') pos++;
        bool leftJustify = false;
        int width = 0;
        if (ch == ',') {
            pos++;
            while (pos < len && format[pos] == ' ') pos++;
 
            if (pos == len) FormatError();
            ch = format[pos];
            if (ch == '-') {
                leftJustify = true;
                pos++;
                if (pos == len) FormatError();
                ch = format[pos];
            }
            if (ch < '0' || ch > '9') FormatError();
            do {
                width = width * 10 + ch - '0';
                pos++;
                if (pos == len) FormatError();
                ch = format[pos];
            } while (ch >= '0' && ch <= '9' && width < 1000000);
        }
 
        while (pos < len && (ch = format[pos]) == ' ') pos++;
        Object arg = args[index];
        StringBuilder fmt = null;
        if (ch == ':') {
            pos++;
            p = pos;
            i = pos;
            while (true) {
                if (pos == len) FormatError();
                ch = format[pos];
                pos++;
                if (ch == '{')
                {
                    if (pos < len && format[pos] == '{')  // Treat as escape character for {{
                        pos++;
                    else
                        FormatError();
                }
                else if (ch == '}')
                {
                    if (pos < len && format[pos] == '}')  // Treat as escape character for }}
                        pos++;
                    else
                    {
                        pos--;
                        break;
                    }
                }
 
                if (fmt == null) {
                    fmt = new StringBuilder();
                }
                fmt.Append(ch);
            }
        }
        if (ch != '}') FormatError();
        pos++;
        String sFmt = null;
        String s = null;
        if (cf != null) {
            if (fmt != null) {
                sFmt = fmt.ToString();
            }
            s = cf.Format(sFmt, arg, provider);
        }
 
        if (s == null) {
            IFormattable formattableArg = arg as IFormattable;
 
#if FEATURE_LEGACYNETCF
            if(CompatibilitySwitches.IsAppEarlierThanWindowsPhone8) {
            // TimeSpan does not implement IFormattable in Mango
            
            if(arg is TimeSpan) {
                formattableArg = null;
                }
            }
#endif
            if (formattableArg != null) {
                if (sFmt == null && fmt != null) {
                    sFmt = fmt.ToString();
                }
 
                s = formattableArg.ToString(sFmt, provider);
            } else if (arg != null) {
                s = arg.ToString();
            }
        }
 
        if (s == null) s = String.Empty;
        int pad = width - s.Length;
        if (!leftJustify && pad > 0) Append(' ', pad);
        Append(s);
        if (leftJustify && pad > 0) Append(' ', pad);
    }
    return this;
}
```

__따라서 Intern Pool과 string.Format 및 string Interpolation은 관계가 없으며 매번 새로운 문자열을 리턴한다.__