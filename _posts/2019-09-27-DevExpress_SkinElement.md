---
title:  "DevExpress에서 Skin 요소 가져오기"
search: true
categories: 
  - DevExpress
tag:
  - C#
  - WinForm
  - DevExpress
  - SkinElement
---

> DevExpress에서는 Skin의 요소를 가져올 수 있습니다.

```C#
Skin skin = GridSkins.GetSkin(DevExpress.LookAndFeel.UserLookAndFeel.Default.ActiveLookAndFeel);
SkinElement elem = skin[GridSkins.SkinGridEvenRow];
```

위 코드는 현재 활성화된 DevExpress 스킨을 가져와서 GridView의 짝수행의 배경색을 가져옵니다.

짝수행의 배경색이란,
```C#
GridView.OptionsView.EnableAppearanceEvenRow = true;
```
GridView.OptionsView.EnableAppearanceEvenRow 프로퍼티를 true로 설정했을 때 GridView의 짝수행에 생기는 색상입니다.

마찬가지로 홀수행도 같은 프로퍼티를 지정할 수 있습니다.

```C#
GridView.OptionsView.EnableAppearanceOddRow = true;
```
GridView.OptionsView.EnableAppearanceOddRow 프로퍼티를 true로 지정하시면 됩니다.

샘플 코드를 이용하여 GridView의 Row 색을 조건부로 변경할 수 있습니다.

아래처럼 GridView RowStyle 이벤트를 생성하시고, 등록하신 뒤,
```C#
GridView.RowStyle += GridView_RowStyle;
```
if 문을 이용해서 

```C#
private void GridView_RowStyle(object sender, RowStyleEventArgs e)
{
  if (GridView.GetRowCellValue(e.RowHandle, "컬럼명 또는 컬럼 객체명"))
  {
      SkinElement elem = GridSkins.GetSkin(DevExpress.LookAndFeel.UserLookAndFeel.Default.ActiveLookAndFeel)[GridSkins.SkinGridEvenRow];
      e.Appearance.BackColor = elem.Color.BackColor;
      e.HighPriority = true;
  }
}
```

위와 같이 코딩하시면 특정 컬럼의 조건에 따라 해당 Row 색상을 변경하실 수 있습니다.
