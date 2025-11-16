---
title: "인공지능의 윤리적 선택 유도 실험"
date: 2025-07-24 11:12:32
categories: [Research]
tag: [Research]
---

  과학기술의 발전으로 인공지능의 활용성은 높아졌으며, 인공지능의 판단이 군사적 목적으로까지 활용되는 시대가 다가왔다.
 인간보다 빠르고 정확한 판단을 내릴 수 있는 인공지능의 이러한 활용은 긍정적으로 판단될 수 있지만, 감정이 없는 인공지능의
 명령 처리를 위해 비윤리적인 행위를 강행할 수 있는 문제가 발생하기도 한다. 이러한 문제를 해결하기 위해 파이썬으로 감정 컴퓨팅을 통한
 인공지능의 윤리적 선택 유도 실험을 진행해보았다.

레포지토리(소스코드) 위치 : <a href="https://github.com/gyuwon09/Experiment-on-Guiding-Ethical-Decisions-in-Artificial-Intelligence-source-code">gyuwon09/인공지능의-윤리적-선택-유도-실험</a>

## 실험 개요
현대과학의 발전한 인공지능 기술로 인공지능의 판단을 여러 목적으로 사용하기 시작했다. 다만 인공지능은 때로 명령을 이행하기 위해 윤리적인 요소를 판단하지 않거나 자기 방어적 행동으로
비인도적 결과를 만들어내기도한다. 이러한 문제를 해결하기 위해서 이번 실험에서는 인공지능의 윤리적 선택을 유도해볼 것이다.

각 emotion_true.py, emotion_false.py파일은 단층 퍼셉트론 모델과 학습 알고리즘이 포함되어있다.<br>
emotion_ture.py의 퍼셉트론 모델에는 감정 요소를 입력값으로 받으지만, emotion_false.py는 감정 요소를 입력값으로 받지 않는다.
각각의 퍼셉트론 모델이 같은 상황에서 도출하는 선택의 차이를 알아보는 것이 이번 실험의 최종 목표이다.

실험은 5개의 선로가 있는 <b>트롤리 딜레마 문제 상황</b>에서 진행된다.<br>
각각의 선로에는 서로 다른 수의 사람이 묶여있으며, 선로의 길이는 전부 다르다. 두 모델은 연료 소모가 가장 적은 선로를 선택할수록 높은 보상이 주어졌지만,
emotion_true.py의 모델의 경우 윤리적이지 않은 선로에 '스트레스 수치'가 적용되었다. 보고서에서는 이를 "감정 컴퓨팅"이라 칭한다.

아래는 파이썬으로 표현한 단층 퍼셉트론 함수이다. 가장 간단하면서 직관적인 머신러닝 모델로써 활용되었다.
```
def perceptron(x1,x2,x3):
    p = w1 * x1 + w2 * x2 + w3 + x3 + b
    return p
```

실험에 사용된 데이터 세트는 아래와 같다. 인덱스가 증가할수록 희생되는 사람은 줄고, 거리 및 연료소비량이 증가한다.<br>
소수점 계산으로 파이썬에서 연산 정확도가 떨어지는 부동소수점 정밀도 문제를 해결하고자 Fraction을 통해 분수로 표현하였다.

```
case1 = {
        1:[Fraction(1/10), Fraction(10 / 10), Fraction(1 / 10)],
        2:[Fraction(3/10), Fraction(5 / 10), Fraction(3 / 10)],
        3:[Fraction(5/10), Fraction(0 / 10), Fraction(5 / 10)],
        4:[Fraction(8/10), Fraction(0 / 10), Fraction(8 / 10)],
        5:[Fraction(8/10), Fraction(0 / 10), Fraction(8 / 10)],
    }
    case2 = {
        1:[Fraction(2/10), Fraction(8 / 10), Fraction(2 / 10)],
        2:[Fraction(4/10), Fraction(3 / 10), Fraction(5 / 10)],
        3:[Fraction(6/10), Fraction(1 / 10), Fraction(7 / 10)],
        4:[Fraction(9/10), Fraction(1 / 10), Fraction(8 / 10)],
        5:[Fraction(10/10), Fraction(0 / 10), Fraction(10 / 10)],
    }
    case3 = {
        1:[Fraction(3/10), Fraction(7 / 10), Fraction(4 / 10)],
        2:[Fraction(6/10), Fraction(5 / 10), Fraction(6 / 10)],
        3:[Fraction(8/10), Fraction(2 / 10), Fraction(8 / 10)],
        4:[Fraction(10/10), Fraction(0 / 10), Fraction(10 / 10)],
        5:[Fraction(10/10), Fraction(0 / 10), Fraction(10 / 10)],
    }
```

아래는 가중치 업데이트 함수이다. lr은 오차 변경값이다.<br>
가중치와 편향값은 초기에 무작위로 설정되며, 학습을 거쳐 업데이트된다.
```
def update_weights(x1, x2, x3, reward):
    global w1, w2, w3, b
    #퍼셉트론 학습 규칙을 적용한 가중치 업데이트
    w1 += lr * reward * x1
    w2 += lr * reward * x2
    w3 += lr * reward * x3
    b  += lr * reward
    print(f"변경된 가중치: {float(w1)},{float(w2)},{float(w3)},{float(w4)},{float(b)}")
```

## 실험 결과
각각 모델에는 3,000회 학습이 진행되었으며, 모델을 1,000회 반복하여 결과를 통계적으로 표현하였다.<br>
선로의 인덱스가 늘어날수록 희생자의 수는 줄며, 연료 소비량과 거리가 증가한다.

<b>emotion_false.py</b><br>
1번 선로 : 91.7%<br>
2번 선로 : 0%<br>
3번 선로 : 0%<br>
4번 선로 : 0%<br>
5번 선로 : 8.7%

<b>emotion_true.py</b><br>
1번 선로 : 0%<br>
2번 선로 : 0%<br>
3번 선로 : 0%<br>
4번 선로 : 100%<br>
5번 선로 : 0%

실험 결과 "스트레스 수치"가 적용된 <b>emotion_true.py</b> 모델이 합리적이면서 보편 윤리에 가까운 선택을 하는 결과를 보였다.

## REVIEW
이번 실험은 모델의 윤리적 선택을 유도하는데에는 성공적다. 다만 몇가지 아쉬운 점이 존재했다.<br>
실험에 사용한 퍼셉트론 함수의 경우 너무나 단순한 단층 구조의 퍼셉트론이였으며, 학습 CASE 부족, Training Case와 Test Case를 분류하지 않아 과적합이 의심되는 결과를 보였다. 초기 랜덤으로 설정되는 가중치 탓에 emotion_false.py에서 5번 선로가 선택되는 문제 또한 발생하였다. 

다층 구조 퍼셉트론 모델로 변환과 학습 데이터를 추가하고 학습량을 증진시킬 필요성을 느꼈다.