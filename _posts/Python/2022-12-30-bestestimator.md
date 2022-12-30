---
layout: single
title: "[Scikit-Learn] 커스텀 변환기 만들기"
excerpt: "sklearn의 bestestimator와 transformaermixin을 활용하여 원하는 기능을 갖춘 커스텀 변환기를 만들어보자"

published: true

categories:
  - Python

tags: [python, Scikit-Learn, class]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2022-12-30 11:45:00
last_modified_at: 2022-12-30 11:45:00
---

<br>

dacon 등의 인공지능 대회에서 코드를 작성하다보면 난잡해지거나, 매번 같은 코드를 실행하다보니 귀찮은 점들이 있다. 특히나 전처리 과정에서 대부분의 코드를 할애하기에 더욱 지저분해지고 시간이 낭비되는 감이 있다.

이러한 문제를 해결하기 위해 pipeline을 사용하곤 한다. 대부분 머신러닝을 할 때 자주 쓰는 변환기로는 StandaradScaler나 MinmaxScaler 등이 있다. 필자는 한 발 더 나아가 원하는 기능을 구현하는 변환기를 직접 만들어 pipeline에 넣어보려 한다.


우선 필요한 모듈을 불러온다.

```python
from sklearn.base import BaseEstimator, TransformerMixin
```

이번 예시에서 활용한 데이터셋은 데이콘 유전체 정보 품종 분류 대회에서 쓰인 데이터이다. 컬럼 목록은 아래와 같고, target이 되는 class 컬럼은 A, B, C의 문자열로 이루어진 멀티클래스이다.

```python
Index(['id', 'father', 'mother', 'gender', 'trait', 'SNP_01', 'SNP_02',
       'SNP_03', 'SNP_04', 'SNP_05', 'SNP_06', 'SNP_07', 'SNP_08', 'SNP_09',
       'SNP_10', 'SNP_11', 'SNP_12', 'SNP_13', 'SNP_14', 'SNP_15', 'class'],
      dtype='object')
```

<br>

만약 특정 원하는 컬럼을 drop하는 변환기를 만들고 싶다면 아래처럼 코드를 작성하면 된다.

```python
class Dropper(BaseEstimator, TransformerMixin):
  def fit(self, X, y=None):
    return self
  
  def transform(self, X):
    cols_drop = ['id','father','mother','gender']
    return X.drop(columns=cols_drop)
```

<br>

또한 만약에 `SNP_01`이란 컬럼에 결측값이 존재하고, 이를 평균값으로 채워넣고 싶다면 아래처럼 작성하면 된다.

```python
class Dropper(BaseEstimator, TransformerMixin):
  def fit(self, X, y=None):
    return self

  def transform(self, X):
    X['SNP_01'].fillna(X['SNP_01'].mean(), inplace=True)
    return X
```

<br>

사이킷런의 `LabelEncoder()`는 y값에 대해서만 변환이 되도록 설계되었기 때문에 1차원에 대해서만 fit이 가능하다. 이 문제를 해결하기 위해 커스텀 변환기를 아래처럼 작성할 수 있다.

```python
class Labelencoder(BaseEstimator, TransformerMixin):
  def fit(self, X, y=None):
    return self

  def transform(self, X):
    snp_le = LabelEncoder()
    snp_col = [f'SNP_{str(x).zfill(2)}' for x in range(1,16)] #SNP로 시작하는 컬럼만 대상으로 함
    for col in snp_col:
      snp_data = []
      snp_data += list(X[col].values)
      snp_le.fit(snp_data)
      X[col] = snp_le.transform(X[col])
    return X
```