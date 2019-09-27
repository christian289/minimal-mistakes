---
title:  "DevExpress를 사용하여 기본 스킨 설정하기"
search: true
toc: true
toc_sticky: true
categories: 
  - DevExpress
tag:
  - C#
  - DevExpress
  - Skin
---

DevExpress에는 UI를 보기 좋게 하기 위해 자체적으로 Skin을 제공하며, Skin 역시 기존 것을 사용하지 않고 Custom으로 제작도 할 수 있습니다.

아래 링크에서 SkinEditor에 대해서 자세히 알아보실 수 있습니다.
[DevExpress SkinEditor](https://documentation.devexpress.com/SkinEditor/1630/WinForms-Skin-Editor)


```cs
UserLookAndFeel.Default.SetSkinStyle("DevExpress 스킨명");
```

위 코드를 프로그램이 시작할 때 적용해주시면 설정한 스킨이 적용된 채로 프로그램이 실행되는 것을 보실 수 있습니다.

DevExpress 18.1 기준 스킨 목록은 아래와 같습니다.
1. DevExpress Style
2. DevExpress Dark Style
3. Office 2016 Colorful
4. Office 2016 Dark
5. Office 2016 Black
6. The Bezier
7. Office 2013 White
8. Office 2013 Dark Gray
9. Office 2013 Light Gray
10. Office 2010 Blue
11. Office 2010 Black
12. Office 2010 Silver
13. Visual Studio 2013 Blue
14. Visual Studio 2013 Dark
15. Visual Studio 2013 Light
16. Seven Classic
17. Visual Studio 2010
18. Black
19. Blue
20. Caramel
21. Coffee
22. Dark Side
23. Darkroom
24. Foggy
25. Glass Oceans
26. High Contrast
27. iMaginary
28. Lilian
29. Liquid Sky
30. London Liquid Sky
31. Metropolis
32. Metropolis Dark
33. Money Twins
34. Office 2017 Black
35. Office 2017 Blue
36. Office 2017 Green
37. Office 2017 Pink
38. Office 2017 Silver
39. Seven
40. Sharp
41. Sharp Plus
42. Stardust
43. The Asphalt World
44. Pumpkin
45. Springtime
46. Summer
47. Valentine
48. Xmas (Blue)
49. McSkin
50. Blueprint
51. Whiteprint

