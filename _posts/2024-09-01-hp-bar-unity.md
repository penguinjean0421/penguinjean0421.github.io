---
layout: post
read_time: true
show_date: true
title: "unity로 Hp 바 구현하기"
date: yyyy-mm-dd 00:00:00 +0900
img: posts/yyyymmdd/
tags: []
category:
author: Penguin Jean
description: "description"
---

## 서론
2024 인디 게임잼 대전에서 게임에 적용헸단  HP 바 로직을 소개한다.

## 1. 배경

### 유니티에서  Hp 바를  구현하는  방법

Unity 엔진에서 Hp를 구현하는 방법은 여러가지가 있다.  많이 사용하는 방법은 Slider를 이용하는 방식인데, 간단 하지만 Slider 1칸 안에 모든 체력을 표현해야 한다. 이러한 방식은 적은 체력을 가지고 있을때는 효과를 주는데 큰 문제가 없지만, 억 / 조 단위 같은 큰수로 넘어오게 되면 상대적으로 데미지가 적은 초창기에는 데미지 깎이고 있다는 인식을 주기가 어려워 다른 방식을 고민하던 중, Image를 이용한 방식이 있는것을 알게되어, Image를 통해 구현하는 방법도 후보에 오르게되었다.

### 5조 를 체력바에 표현하는 방법
10의 12제곱인 조 라는 숫자를 체력바 1개로만 구현하려 한다면 데미지가 상대적으로 적게 들어가는 초반부에는 현미경으로 봐야 겨우 보일 정도로 미세할 것이다. 시각적인 효과를 위해 hp바를 일정하게 나누고, 나눠진 바마다 색상을 다르게 하는 방식을 채택했다. 

색상은 5가지 정도 사용했는데, 다양한 색상을 사용 할 수 있지만, 래퍼런스로 참고했던 게임들이 보스 체력바 색상을 5개 내외로 사용했기에 5가지로 제한했다. 시각적인 것이 중요했던 만큼 색상 선정과 배치는 무지개 색상 중에 남색과 주황을 제외한 색의 역순(보라 > 파랑 > 초록 > 노랑 > 빨강)으로 배치를 하였다.

## 2. 본격적인 구현

1에서 설명한 배경을 바탕으로, Image 방식을 채택해 Hp바를 구현하였다.

```c#
public Image CurrHpLong, NextHpLong; // 현재 체력바 이미지와 다음 체력바 이미지

public long hpSingleBar; // 단위 체력바당 체력

public long currHp; // 현재 체력
public long maxHp; // 최대 체력

public List<Color> colors; // 색상 리스트
```
구현에서 사용될 변수들을 선언했다. 5조는 정수 자료형인 int의 범위인 -21억 ~ 21억 범위를 넘는다. 그래서 lomg 자료형으로 변수를 선언 하게 된다.
```c#
private void Awake()
{
	maxHp = 5000000000000;
	currHp = 5000000000000;
	hpSingleBar = 10000000000;
 }
```
설정한 변수 안에 체력을 입력한다. 최대 체력은 5조, 단위 체력바당 체력은 10억으로 설정했다.
```c#
void Refresh()
{
	CurrHpLong.rectTransform.sizeDelta = new Vector2(NextHpLong.rectTransform.sizeDelta.x * GetHpRationSingleBar(currHp), NextHpLong.rectTransform.sizeDelta.y);
	CurrHpLong.color = GetColorByLayer(currHp);
    NextHpLong.color = GetColorByLayer(currHp - hpSingleBar);
}
```
ReFresh 함수는 체력의 변화에 따라 체력 바가 업데이트 할 수 있게 해주는 함수다.  체력이 줄어들게 되면 체력바의 x값이 줄어든다.
```c#
float GetHpRationSingleBar(long lnTargetHP)
    {
        float fResultRatio = 0.0f;

        float fDivided = (float)lnTargetHP / hpSingleBar;

        if (fDivided == (int)fDivided)
        {
            fResultRatio = 1.0f;
        }
        else
        {
            float moudled = lnTargetHP % hpSingleBar;

            fResultRatio = moudled / hpSingleBar;
        }

        return fResultRatio;
    }
```
단위 체력바당표현되는 체력을 표현해주는 함수입니다.
```c#
    Color GetColorByLayer(long nTargetHP)
    {
        Color result = Color.black;

        if (nTargetHP > 0)
        {
            float fDivided = (float)nTargetHP / hpSingleBar;

            long index = (long)fDivided;

            if (fDivided == (long)fDivided)
            {
                index--;
            }

            result = colors[(int)index % colors.Count];
        }

        return result;
    }
```
남은 HP에 맞춰 HP바의 색상을 바꿔주는 함수 입니다.  
