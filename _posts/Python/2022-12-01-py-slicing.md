---
layout: single
title: "[Python] 판다스(pandas) 데이터프레임(DataFrame)에서 특정 열의 문자열을 슬라이싱하기[feat. map(lambda x)]"
excerpt: "판다스 데이터프레임 형식에서 특정 열의 문자열을 슬라이싱하고 이를 새로운 열로 추가해보자"

published: true

categories:
  - Python

tags: [python, pandas]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2022-12-01 10:50:00
last_modified_at: 2022-12-01 10:50:00
---

<br>




파이썬에서 문자열을 슬라이싱(slicing)하는 것은 간단하지만, 판다스(pandas)에서 데이터프레임의 특정 열을 슬라이싱하는 것은 번거로울 수 있다. 이를테면 아래와 같은 상황이다.

```python
result
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>선수</th>
      <th>G</th>
      <th>타석</th>
      <th>타수</th>
      <th>득점</th>
      <th>안타</th>
      <th>2타</th>
      <th>3타</th>
      <th>홈런</th>
      <th>루타</th>
      <th>...</th>
      <th>타율</th>
      <th>출루율</th>
      <th>장타율</th>
      <th>OPS</th>
      <th>wOBA</th>
      <th>wRC+</th>
      <th>WAR+</th>
      <th>WPA</th>
      <th>생년</th>
      <th>포지션</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>이정후22키CF</td>
      <td>142</td>
      <td>627</td>
      <td>553</td>
      <td>85</td>
      <td>193</td>
      <td>36</td>
      <td>10</td>
      <td>23</td>
      <td>318</td>
      <td>...</td>
      <td>.349</td>
      <td>.421</td>
      <td>.575</td>
      <td>.996</td>
      <td>.441</td>
      <td>182.5</td>
      <td>9.23</td>
      <td>6.72</td>
      <td>1998</td>
      <td>CF</td>
    </tr>
    <tr>
      <th>1</th>
      <td>피렐라22삼LF</td>
      <td>141</td>
      <td>630</td>
      <td>561</td>
      <td>102</td>
      <td>192</td>
      <td>33</td>
      <td>4</td>
      <td>28</td>
      <td>317</td>
      <td>...</td>
      <td>.342</td>
      <td>.411</td>
      <td>.565</td>
      <td>.976</td>
      <td>.434</td>
      <td>169.3</td>
      <td>7.40</td>
      <td>4.20</td>
      <td>1989</td>
      <td>LF</td>
    </tr>
    <tr>
      <th>2</th>
      <td>나성범22KRF</td>
      <td>144</td>
      <td>649</td>
      <td>563</td>
      <td>92</td>
      <td>180</td>
      <td>39</td>
      <td>2</td>
      <td>21</td>
      <td>286</td>
      <td>...</td>
      <td>.320</td>
      <td>.402</td>
      <td>.508</td>
      <td>.910</td>
      <td>.411</td>
      <td>157.4</td>
      <td>6.50</td>
      <td>3.74</td>
      <td>1989</td>
      <td>RF</td>
    </tr>
    <tr>
      <th>3</th>
      <td>오지환22LSS</td>
      <td>142</td>
      <td>569</td>
      <td>494</td>
      <td>75</td>
      <td>133</td>
      <td>16</td>
      <td>4</td>
      <td>25</td>
      <td>232</td>
      <td>...</td>
      <td>.269</td>
      <td>.357</td>
      <td>.470</td>
      <td>.827</td>
      <td>.372</td>
      <td>138.6</td>
      <td>5.77</td>
      <td>2.56</td>
      <td>1990</td>
      <td>SS</td>
    </tr>
    <tr>
      <th>4</th>
      <td>최정22S3B</td>
      <td>121</td>
      <td>505</td>
      <td>414</td>
      <td>80</td>
      <td>110</td>
      <td>21</td>
      <td>0</td>
      <td>26</td>
      <td>209</td>
      <td>...</td>
      <td>.266</td>
      <td>.386</td>
      <td>.505</td>
      <td>.891</td>
      <td>.400</td>
      <td>145.4</td>
      <td>5.15</td>
      <td>3.72</td>
      <td>1987</td>
      <td>3B</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>95</th>
      <td>고종욱22KDH</td>
      <td>62</td>
      <td>114</td>
      <td>106</td>
      <td>13</td>
      <td>30</td>
      <td>7</td>
      <td>1</td>
      <td>2</td>
      <td>45</td>
      <td>...</td>
      <td>.283</td>
      <td>.327</td>
      <td>.425</td>
      <td>.752</td>
      <td>.341</td>
      <td>110.4</td>
      <td>0.40</td>
      <td>0.63</td>
      <td>1989</td>
      <td>DH</td>
    </tr>
    <tr>
      <th>96</th>
      <td>김기환22NCF</td>
      <td>73</td>
      <td>131</td>
      <td>113</td>
      <td>26</td>
      <td>25</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>32</td>
      <td>...</td>
      <td>.221</td>
      <td>.310</td>
      <td>.283</td>
      <td>.593</td>
      <td>.282</td>
      <td>71.0</td>
      <td>0.37</td>
      <td>-1.17</td>
      <td>1995</td>
      <td>CF</td>
    </tr>
    <tr>
      <th>97</th>
      <td>김주형22키SS</td>
      <td>55</td>
      <td>130</td>
      <td>110</td>
      <td>16</td>
      <td>22</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>34</td>
      <td>...</td>
      <td>.200</td>
      <td>.305</td>
      <td>.309</td>
      <td>.614</td>
      <td>.291</td>
      <td>76.0</td>
      <td>0.33</td>
      <td>-0.93</td>
      <td>1996</td>
      <td>SS</td>
    </tr>
    <tr>
      <th>98</th>
      <td>박상언22한C</td>
      <td>56</td>
      <td>151</td>
      <td>134</td>
      <td>16</td>
      <td>30</td>
      <td>5</td>
      <td>1</td>
      <td>4</td>
      <td>49</td>
      <td>...</td>
      <td>.224</td>
      <td>.281</td>
      <td>.366</td>
      <td>.647</td>
      <td>.287</td>
      <td>73.4</td>
      <td>0.32</td>
      <td>-0.83</td>
      <td>1997</td>
      <td>C</td>
    </tr>
    <tr>
      <th>99</th>
      <td>안승한22두C</td>
      <td>30</td>
      <td>39</td>
      <td>36</td>
      <td>5</td>
      <td>12</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>15</td>
      <td>...</td>
      <td>.333</td>
      <td>.368</td>
      <td>.417</td>
      <td>.785</td>
      <td>.356</td>
      <td>122.1</td>
      <td>0.32</td>
      <td>0.21</td>
      <td>1992</td>
      <td>C</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 30 columns</p>
</div>

위에 result라는 데이터프레임에서 '선수'라는 열은 '선수이름+시즌연도+팀(1글자)+포지션'으로 구성되어있다. 여기서 선수이름만 추출하여 새로운 열로 만들고 싶은 상황에 어떻게 해야 할까?

```python
result['선수'][:3]
```
```
0    이정후22키CF
1    피렐라22삼LF
2    나성범22KRF
Name: 선수, dtype: object
```

위같은 코드를 작성하면 문자열이 슬라이싱되는 것이 아니라, 1~3행이 출력되어 버린다.

<br>

그래서 데이터프레임을 다루는 것이 번거롭다는 얘기다. 하지만 아래같이 작성하면 문제가 해결된다.

```python
result['선수'].str.slice(0,3)
```




    0     이정후
    1     피렐라
    2     나성범
    3     오지환
    4     최정2
         ... 
    95    고종욱
    96    김기환
    97    김주형
    98    박상언
    99    안승한
    Name: 선수, Length: 100, dtype: object


보통 한국 사람의 이름은 3글자이니까, slice를 0부터 3의 값으로 설정했다. 그러나 이 역시 문제가 존재한다. 5행의 최정 선수 같은 외자인 이름은 제대로 슬라이싱되지 않는다. 더군다나 외국인 선수의 경우 이름이 2~5글자 등 판이할 수 있다.

<br>

그럴 때 `map`과 `lambda`를 적절히 사용해주면 된다. 여기선 `find`함수도 쓰였는데, `find`는 문자열 내에서 찾고자 하는 문자가 어느 위치(index)인지를 반환해준다.

```python
문자열.find('찾고자 하는 문자')
```

즉, 아래 예시는 선수 이름 뒤에 오는 시즌년도(22)의 위치를 찾고, 그 위치전까지만 출력을 하겠다는 의미이다.
```python
result['선수'].map(lambda x:x[:x.find('22')])
```




    0     이정후
    1     피렐라
    2     나성범
    3     오지환
    4      최정
         ... 
    95    고종욱
    96    김기환
    97    김주형
    98    박상언
    99    안승한
    Name: 선수, Length: 100, dtype: object


<br>

이를 응용하여 선수의 포지션만 출력하고 싶다면, 시즌년도에 +3을 하여 포지션이 나올 인덱스를 적절히 설정해주면 될 것이다!

```python
result['선수'].map(lambda x:x[x.find('22')+3:])
```




    0     CF
    1     LF
    2     RF
    3     SS
    4     3B
          ..
    95    DH
    96    CF
    97    SS
    98     C
    99     C
    Name: 선수, Length: 100, dtype: object


마지막으로, 포지션을 추출한 값을 '포지션'이라는 이름의 새로운 열로 생성하려면 아래처럼 작성해주면 된다!

```python
result['포지션'] = result['선수'].map(lambda x:x[x.find('22')+3:])
```

<br>
