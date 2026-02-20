---
title: "Random Forest 모델을 이용한 붓꽃(Iris) 분류 탐구"
date: 2026-02-17 20:27:13
categories: [Research]
tag: [Research]
---
scikit-learn 머신러닝 라이브러리를 활용하여
머신러닝의 대표적인 문제인 붓꽃 분류에 대해 탐구해보았습니다.

붓꽃(Iris) 분류는 세 가지 품종의 붓꽃을 분류하는 문제를 다룹니다.
붓꽃 데이터셋에는 꽃받침 길이(sepal length), 꽃받침 너비(sepal width), 꽃잎 길이(petal length), 꽃잎 너비(petal width)와 같은
특성이 있으며, 이를 기반으로 세 가지 품종으로 구분이 가능합니다.

## 
```
from sklearn.datasets import load_iris
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

iris = load_iris()
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['target'] = iris.target

sns.pairplot(df, hue='target')
plt.show()
```