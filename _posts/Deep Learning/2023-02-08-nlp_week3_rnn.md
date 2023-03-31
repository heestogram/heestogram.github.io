---
title: "[DL] RNN을 활용한 주가 예측"
excerpt: "RNN을 사용하여 카카오 주가를 예측해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, RNN]

published: true

categories:
  - DL

date: 2023-02-08 21:00:00
last_modified_at: 2023-02-08 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import warnings
warnings.filterwarnings("ignore")
```

<br>

## 1. finance-datareader로 주가 데이터 갖고 오기
finance-datareader는 한국, 미국 주식 가격, 지수, 환율 등 금융 데이터를 수집해놓은 라이브러리이다.

`DataRead(종목코드, 시작날짜, 종료날짜)`로 주가 데이터를 갖고올 수 있다.


```python
!pip install finance-datareader
```


```python
import FinanceDataReader as fdr
```


```python
# 2016년~2022년 카카오 주가 데이터
data = fdr.DataReader('035720', start='20160101', end='20221231')
```


```python
data.head()
```





  <div id="df-020c8f0c-2622-4040-8259-82d21369c2e1">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-01-04</th>
      <td>23583</td>
      <td>23583</td>
      <td>23142</td>
      <td>23142</td>
      <td>297687</td>
      <td>-0.004345</td>
    </tr>
    <tr>
      <th>2016-01-05</th>
      <td>22900</td>
      <td>23583</td>
      <td>22840</td>
      <td>23503</td>
      <td>318036</td>
      <td>0.015599</td>
    </tr>
    <tr>
      <th>2016-01-06</th>
      <td>23684</td>
      <td>24306</td>
      <td>23543</td>
      <td>23905</td>
      <td>544137</td>
      <td>0.017104</td>
    </tr>
    <tr>
      <th>2016-01-07</th>
      <td>23764</td>
      <td>24186</td>
      <td>23403</td>
      <td>23544</td>
      <td>342194</td>
      <td>-0.015101</td>
    </tr>
    <tr>
      <th>2016-01-08</th>
      <td>23142</td>
      <td>23262</td>
      <td>22820</td>
      <td>23122</td>
      <td>400046</td>
      <td>-0.017924</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-020c8f0c-2622-4040-8259-82d21369c2e1')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-020c8f0c-2622-4040-8259-82d21369c2e1 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-020c8f0c-2622-4040-8259-82d21369c2e1');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
data.tail()
```





  <div id="df-8475c9c0-5d81-44ee-985d-0e21a239e8f7">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2022-12-23</th>
      <td>54000</td>
      <td>54300</td>
      <td>53100</td>
      <td>53400</td>
      <td>1339673</td>
      <td>-0.030853</td>
    </tr>
    <tr>
      <th>2022-12-26</th>
      <td>53400</td>
      <td>53800</td>
      <td>52700</td>
      <td>53600</td>
      <td>988777</td>
      <td>0.003745</td>
    </tr>
    <tr>
      <th>2022-12-27</th>
      <td>53900</td>
      <td>54700</td>
      <td>53600</td>
      <td>54400</td>
      <td>1226474</td>
      <td>0.014925</td>
    </tr>
    <tr>
      <th>2022-12-28</th>
      <td>53900</td>
      <td>54700</td>
      <td>52900</td>
      <td>53600</td>
      <td>1268005</td>
      <td>-0.014706</td>
    </tr>
    <tr>
      <th>2022-12-29</th>
      <td>53500</td>
      <td>55300</td>
      <td>52900</td>
      <td>53100</td>
      <td>1319611</td>
      <td>-0.009328</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-8475c9c0-5d81-44ee-985d-0e21a239e8f7')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-8475c9c0-5d81-44ee-985d-0e21a239e8f7 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-8475c9c0-5d81-44ee-985d-0e21a239e8f7');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>




index로 지정되어있는 Date를 컬럼으로 바꿔준다.


```python
data.reset_index(drop=False, inplace=True)
```


```python
data
```





  <div id="df-20fab1cb-ed07-4916-92d0-e364c2b30535">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-01-04</td>
      <td>23583</td>
      <td>23583</td>
      <td>23142</td>
      <td>23142</td>
      <td>297687</td>
      <td>-0.004345</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-01-05</td>
      <td>22900</td>
      <td>23583</td>
      <td>22840</td>
      <td>23503</td>
      <td>318036</td>
      <td>0.015599</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-01-06</td>
      <td>23684</td>
      <td>24306</td>
      <td>23543</td>
      <td>23905</td>
      <td>544137</td>
      <td>0.017104</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-01-07</td>
      <td>23764</td>
      <td>24186</td>
      <td>23403</td>
      <td>23544</td>
      <td>342194</td>
      <td>-0.015101</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-01-08</td>
      <td>23142</td>
      <td>23262</td>
      <td>22820</td>
      <td>23122</td>
      <td>400046</td>
      <td>-0.017924</td>
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
    </tr>
    <tr>
      <th>1716</th>
      <td>2022-12-23</td>
      <td>54000</td>
      <td>54300</td>
      <td>53100</td>
      <td>53400</td>
      <td>1339673</td>
      <td>-0.030853</td>
    </tr>
    <tr>
      <th>1717</th>
      <td>2022-12-26</td>
      <td>53400</td>
      <td>53800</td>
      <td>52700</td>
      <td>53600</td>
      <td>988777</td>
      <td>0.003745</td>
    </tr>
    <tr>
      <th>1718</th>
      <td>2022-12-27</td>
      <td>53900</td>
      <td>54700</td>
      <td>53600</td>
      <td>54400</td>
      <td>1226474</td>
      <td>0.014925</td>
    </tr>
    <tr>
      <th>1719</th>
      <td>2022-12-28</td>
      <td>53900</td>
      <td>54700</td>
      <td>52900</td>
      <td>53600</td>
      <td>1268005</td>
      <td>-0.014706</td>
    </tr>
    <tr>
      <th>1720</th>
      <td>2022-12-29</td>
      <td>53500</td>
      <td>55300</td>
      <td>52900</td>
      <td>53100</td>
      <td>1319611</td>
      <td>-0.009328</td>
    </tr>
  </tbody>
</table>
<p>1721 rows × 7 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-20fab1cb-ed07-4916-92d0-e364c2b30535')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-20fab1cb-ed07-4916-92d0-e364c2b30535 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-20fab1cb-ed07-4916-92d0-e364c2b30535');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1721 entries, 0 to 1720
    Data columns (total 7 columns):
     #   Column  Non-Null Count  Dtype         
    ---  ------  --------------  -----         
     0   Date    1721 non-null   datetime64[ns]
     1   Open    1721 non-null   int64         
     2   High    1721 non-null   int64         
     3   Low     1721 non-null   int64         
     4   Close   1721 non-null   int64         
     5   Volume  1721 non-null   int64         
     6   Change  1721 non-null   float64       
    dtypes: datetime64[ns](1), float64(1), int64(5)
    memory usage: 94.2 KB
    

<br>

## 2. train, val set 나누기

평가에 사용할 validation set은 전체 데이터의 30%를 사용할 것이다.


```python
length_data = len(data)     # data 행 개수
split_ratio = 0.7           # 0.7 / 0.3 으로 분리
length_train = round(length_data * split_ratio)  
length_validation = length_data - length_train
print("Data length :", length_data)
print("Train data length :", length_train)
print("Validation data lenth :", length_validation)
```

    Data length : 1721
    Train data length : 1205
    Validation data lenth : 516
    


```python
train_data = data[:length_train]
train_data
```





  <div id="df-6840be7c-9405-4769-b9ac-de4b2f8be625">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-01-04</td>
      <td>23583</td>
      <td>23583</td>
      <td>23142</td>
      <td>23142</td>
      <td>297687</td>
      <td>-0.004345</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-01-05</td>
      <td>22900</td>
      <td>23583</td>
      <td>22840</td>
      <td>23503</td>
      <td>318036</td>
      <td>0.015599</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-01-06</td>
      <td>23684</td>
      <td>24306</td>
      <td>23543</td>
      <td>23905</td>
      <td>544137</td>
      <td>0.017104</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-01-07</td>
      <td>23764</td>
      <td>24186</td>
      <td>23403</td>
      <td>23544</td>
      <td>342194</td>
      <td>-0.015101</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-01-08</td>
      <td>23142</td>
      <td>23262</td>
      <td>22820</td>
      <td>23122</td>
      <td>400046</td>
      <td>-0.017924</td>
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
    </tr>
    <tr>
      <th>1200</th>
      <td>2020-11-23</td>
      <td>73964</td>
      <td>74064</td>
      <td>73261</td>
      <td>73663</td>
      <td>363196</td>
      <td>0.004103</td>
    </tr>
    <tr>
      <th>1201</th>
      <td>2020-11-24</td>
      <td>73562</td>
      <td>75670</td>
      <td>73562</td>
      <td>74867</td>
      <td>649755</td>
      <td>0.016345</td>
    </tr>
    <tr>
      <th>1202</th>
      <td>2020-11-25</td>
      <td>75770</td>
      <td>75770</td>
      <td>73462</td>
      <td>73663</td>
      <td>703279</td>
      <td>-0.016082</td>
    </tr>
    <tr>
      <th>1203</th>
      <td>2020-11-26</td>
      <td>73763</td>
      <td>75168</td>
      <td>73562</td>
      <td>75068</td>
      <td>539570</td>
      <td>0.019073</td>
    </tr>
    <tr>
      <th>1204</th>
      <td>2020-11-27</td>
      <td>75369</td>
      <td>75770</td>
      <td>74465</td>
      <td>74867</td>
      <td>335836</td>
      <td>-0.002678</td>
    </tr>
  </tbody>
</table>
<p>1205 rows × 7 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-6840be7c-9405-4769-b9ac-de4b2f8be625')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-6840be7c-9405-4769-b9ac-de4b2f8be625 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-6840be7c-9405-4769-b9ac-de4b2f8be625');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
validation_data = data[length_train:]
validation_data
```





  <div id="df-a189f608-0737-4785-ae03-594babdab762">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1205</th>
      <td>2020-11-30</td>
      <td>74865</td>
      <td>75066</td>
      <td>73862</td>
      <td>73863</td>
      <td>544990</td>
      <td>-0.013410</td>
    </tr>
    <tr>
      <th>1206</th>
      <td>2020-12-01</td>
      <td>74164</td>
      <td>75268</td>
      <td>74064</td>
      <td>75168</td>
      <td>452723</td>
      <td>0.017668</td>
    </tr>
    <tr>
      <th>1207</th>
      <td>2020-12-02</td>
      <td>75369</td>
      <td>75369</td>
      <td>74465</td>
      <td>74867</td>
      <td>440500</td>
      <td>-0.004004</td>
    </tr>
    <tr>
      <th>1208</th>
      <td>2020-12-03</td>
      <td>74867</td>
      <td>75068</td>
      <td>74064</td>
      <td>75068</td>
      <td>464371</td>
      <td>0.002685</td>
    </tr>
    <tr>
      <th>1209</th>
      <td>2020-12-04</td>
      <td>75068</td>
      <td>78781</td>
      <td>74465</td>
      <td>78179</td>
      <td>1561745</td>
      <td>0.041442</td>
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
    </tr>
    <tr>
      <th>1716</th>
      <td>2022-12-23</td>
      <td>54000</td>
      <td>54300</td>
      <td>53100</td>
      <td>53400</td>
      <td>1339673</td>
      <td>-0.030853</td>
    </tr>
    <tr>
      <th>1717</th>
      <td>2022-12-26</td>
      <td>53400</td>
      <td>53800</td>
      <td>52700</td>
      <td>53600</td>
      <td>988777</td>
      <td>0.003745</td>
    </tr>
    <tr>
      <th>1718</th>
      <td>2022-12-27</td>
      <td>53900</td>
      <td>54700</td>
      <td>53600</td>
      <td>54400</td>
      <td>1226474</td>
      <td>0.014925</td>
    </tr>
    <tr>
      <th>1719</th>
      <td>2022-12-28</td>
      <td>53900</td>
      <td>54700</td>
      <td>52900</td>
      <td>53600</td>
      <td>1268005</td>
      <td>-0.014706</td>
    </tr>
    <tr>
      <th>1720</th>
      <td>2022-12-29</td>
      <td>53500</td>
      <td>55300</td>
      <td>52900</td>
      <td>53100</td>
      <td>1319611</td>
      <td>-0.009328</td>
    </tr>
  </tbody>
</table>
<p>516 rows × 7 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-a189f608-0737-4785-ae03-594babdab762')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-a189f608-0737-4785-ae03-594babdab762 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-a189f608-0737-4785-ae03-594babdab762');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>




분석에 쓰일 것은 시가이므로 Open 컬럼의 값들만 가져온다.


```python
dataset_train = train_data.Open.values # open 컬럼에 있는 변수만 가져오기
dataset_train.shape
```




    (1205,)




```python
# 1차원 -> 2차원 데이터로 변환
# Changing shape from (1205,) to (1205,1)
dataset_train = np.reshape(dataset_train, (-1,1))
dataset_train.shape
```




    (1205, 1)



validation set도 마찬가지의 과정을 거쳐준다.


```python
dataset_validation = validation_data.Open.values  
dataset_validation = np.reshape(dataset_validation, (-1,1))  
```

<br>

## 3. Scaling
데이터의 전체적인 분포를 생각했을 때 정규화를 시켜줘야 한다. `MinMaxScaler()`를 사용하면 feature의 값이 0~1의 범위를 갖도록 스케일링된다.


```python
data.describe()
```





  <div id="df-696c887a-9a9b-4d54-85c2-ff9d2c92f3b1">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1721.000000</td>
      <td>1721.000000</td>
      <td>1721.000000</td>
      <td>1721.000000</td>
      <td>1.721000e+03</td>
      <td>1721.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>49057.984311</td>
      <td>49821.862289</td>
      <td>48313.374201</td>
      <td>49232.969204</td>
      <td>1.150001e+06</td>
      <td>0.000746</td>
    </tr>
    <tr>
      <th>std</th>
      <td>37993.093979</td>
      <td>38595.746965</td>
      <td>37402.362602</td>
      <td>38001.443308</td>
      <td>1.598395e+06</td>
      <td>0.023144</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>14311.000000</td>
      <td>0.000000e+00</td>
      <td>-0.100649</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>21075.000000</td>
      <td>21474.000000</td>
      <td>20773.000000</td>
      <td>21075.000000</td>
      <td>3.417420e+05</td>
      <td>-0.012584</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>27698.000000</td>
      <td>28199.000000</td>
      <td>27296.000000</td>
      <td>27799.000000</td>
      <td>6.295670e+05</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>74164.000000</td>
      <td>75068.000000</td>
      <td>73200.000000</td>
      <td>74064.000000</td>
      <td>1.380887e+06</td>
      <td>0.012292</td>
    </tr>
    <tr>
      <th>max</th>
      <td>172000.000000</td>
      <td>173000.000000</td>
      <td>161000.000000</td>
      <td>169500.000000</td>
      <td>1.889515e+07</td>
      <td>0.155512</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-696c887a-9a9b-4d54-85c2-ff9d2c92f3b1')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-696c887a-9a9b-4d54-85c2-ff9d2c92f3b1 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-696c887a-9a9b-4d54-85c2-ff9d2c92f3b1');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>




주가가 0인 경우가 있어 살펴보니 21년 4월 12-14일이다. 기사를 검색해보니 21년 4월 15일 카카오의 액면분할이 있있어 12~14일 매매 정지가 있었다고 한다.


```python
data[data['Open']==0]
```





  <div id="df-e57e73a5-bb3e-4b29-92aa-5af5498383d4">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1294</th>
      <td>2021-04-12</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>112000</td>
      <td>0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1295</th>
      <td>2021-04-13</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>112000</td>
      <td>0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1296</th>
      <td>2021-04-14</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>112000</td>
      <td>0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-e57e73a5-bb3e-4b29-92aa-5af5498383d4')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-e57e73a5-bb3e-4b29-92aa-5af5498383d4 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-e57e73a5-bb3e-4b29-92aa-5af5498383d4');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler(feature_range = (0,1)) # min - max scaling 

# scaling dataset
dataset_train_scaled = scaler.fit_transform(dataset_train)
scaled_dataset_validation =  scaler.transform(dataset_validation)
```


```python
plt.subplots(figsize = (15,6))
plt.plot(dataset_train_scaled)
plt.xlabel("Days")
plt.ylabel("Open Price")
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229076648-ce5e3c17-0d3a-4c1c-9788-7756d51c204b.png">
    

<br>

## 4. time step 50으로 X_train, y_train 생성
RNN 모델을 만들 때 예측에 쓰이는 관측치를 만들어줘야 한다. 만약 1-50일째의 데이터를 가지고 51일째를 예측하고 싶다면, 이 50일간의 데이터가 관측치이고 `time_step`은 50이 된다. 그리고 2-51일째의 데이터를 가지고 52일째를 예측하게 된다. 즉, 한 칸(하루)씩 이동하면서 예측을 반복하는 것이다.


```python
X_train = []
y_train = []

time_step = 50 # 그 다음 값을 예측하기 위해(51번째) 50개의 데이터 사용

for i in range(time_step, length_train): # 한칸씩 이동하면서 계속 다음 값 예측 반복
    X_train.append(dataset_train_scaled[i-time_step:i,0])
    y_train.append(dataset_train_scaled[i,0])
    
# convert list to array
X_train, y_train = np.array(X_train), np.array(y_train)
```


```python
X_train = np.reshape(X_train, (X_train.shape[0], X_train.shape[1],1))
y_train = np.reshape(y_train, (y_train.shape[0],1))
```


```python
X_test = []
y_test = []

for i in range(time_step, length_validation):
    X_test.append(scaled_dataset_validation[i-time_step:i,0])
    y_test.append(scaled_dataset_validation[i,0])

X_test, y_test = np.array(X_test), np.array(y_test)
```


```python
X_test = np.reshape(X_test, (X_test.shape[0],X_test.shape[1],1)) 
y_test = np.reshape(y_test, (-1,1)) 
```

<br>

## 5. RNN 모델 생성


```python
# importing libraries
from keras.models import Sequential
from keras.layers import Dense
from keras.layers import SimpleRNN
from keras.layers import Dropout

# RNN 초기화
regressor = Sequential()

# 첫번째 RNN 계층 추가 + dropout regulatization
regressor.add(
    SimpleRNN(units = 50, 
              activation = "tanh", 
              return_sequences = True, 
              input_shape = (X_train.shape[1],1))
             )

regressor.add(
    Dropout(0.2)
             )


# 2번째 RNN 계층 추가

regressor.add(
    SimpleRNN(units = 50, 
              activation = "tanh", 
              return_sequences = True)
             )

regressor.add(
    Dropout(0.2)
             )

# 3번째 RNN 계층 추가

regressor.add(
    SimpleRNN(units = 50, 
              activation = "tanh", 
              return_sequences = True)
             )

regressor.add(
    Dropout(0.2)
             )

# 4번째 RNN 계층 추가

regressor.add(
    SimpleRNN(units = 50)
             )

regressor.add(
    Dropout(0.2)
             )

# 출력층 계층 추가
regressor.add(Dense(units = 1))

# compiling RNN
regressor.compile(
    optimizer = "adam", 
    loss = "mean_squared_error",
    metrics = ["accuracy"])

# fitting the RNN # 배치 크기 = 32, epoch = 50
history = regressor.fit(X_train, y_train, epochs = 50, batch_size = 32)
```

    Epoch 1/50
    37/37 [==============================] - 5s 42ms/step - loss: 0.3659 - accuracy: 0.0017
    Epoch 2/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.2305 - accuracy: 8.6580e-04
    Epoch 3/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.1752 - accuracy: 0.0017
    Epoch 4/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.1480 - accuracy: 0.0017
    Epoch 5/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.1032 - accuracy: 0.0017
    Epoch 6/50
    37/37 [==============================] - 2s 44ms/step - loss: 0.0864 - accuracy: 8.6580e-04
    Epoch 7/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0635 - accuracy: 8.6580e-04
    Epoch 8/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0447 - accuracy: 8.6580e-04
    Epoch 9/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0330 - accuracy: 0.0017
    Epoch 10/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0309 - accuracy: 8.6580e-04
    Epoch 11/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0273 - accuracy: 0.0017
    Epoch 12/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0207 - accuracy: 0.0017
    Epoch 13/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0179 - accuracy: 0.0017
    Epoch 14/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0174 - accuracy: 0.0017
    Epoch 15/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0158 - accuracy: 0.0017
    Epoch 16/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0138 - accuracy: 0.0017
    Epoch 17/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0132 - accuracy: 0.0017
    Epoch 18/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0123 - accuracy: 0.0017
    Epoch 19/50
    37/37 [==============================] - 2s 44ms/step - loss: 0.0093 - accuracy: 0.0017
    Epoch 20/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0102 - accuracy: 0.0017
    Epoch 21/50
    37/37 [==============================] - 2s 62ms/step - loss: 0.0085 - accuracy: 0.0017
    Epoch 22/50
    37/37 [==============================] - 3s 74ms/step - loss: 0.0071 - accuracy: 0.0017
    Epoch 23/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0078 - accuracy: 0.0017
    Epoch 24/50
    37/37 [==============================] - 2s 44ms/step - loss: 0.0084 - accuracy: 0.0017
    Epoch 25/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0068 - accuracy: 0.0017
    Epoch 26/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0070 - accuracy: 0.0017
    Epoch 27/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0056 - accuracy: 0.0017
    Epoch 28/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0062 - accuracy: 0.0017
    Epoch 29/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0055 - accuracy: 0.0017
    Epoch 30/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0058 - accuracy: 0.0017
    Epoch 31/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0053 - accuracy: 0.0017
    Epoch 32/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0044 - accuracy: 0.0017
    Epoch 33/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0048 - accuracy: 0.0017
    Epoch 34/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0045 - accuracy: 0.0017
    Epoch 35/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0045 - accuracy: 0.0017
    Epoch 36/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0048 - accuracy: 0.0017
    Epoch 37/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0033 - accuracy: 0.0017
    Epoch 38/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0044 - accuracy: 0.0017
    Epoch 39/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0039 - accuracy: 0.0017
    Epoch 40/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0042 - accuracy: 0.0017
    Epoch 41/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0037 - accuracy: 0.0017
    Epoch 42/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0031 - accuracy: 0.0017
    Epoch 43/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0037 - accuracy: 0.0017
    Epoch 44/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0036 - accuracy: 0.0017
    Epoch 45/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0035 - accuracy: 0.0017
    Epoch 46/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0031 - accuracy: 0.0017
    Epoch 47/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0033 - accuracy: 0.0017
    Epoch 48/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0032 - accuracy: 0.0017
    Epoch 49/50
    37/37 [==============================] - 2s 43ms/step - loss: 0.0028 - accuracy: 0.0017
    Epoch 50/50
    37/37 [==============================] - 2s 42ms/step - loss: 0.0029 - accuracy: 0.0017
    


```python
plt.figure(figsize =(10,7))
plt.plot(history.history["loss"])
plt.xlabel("Epochs")
plt.ylabel("Losses")
plt.title("Simple RNN model")
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229076761-f835e45d-17f1-4012-a2b7-acc443d19683.png">
    



```python
plt.figure(figsize =(10,5))
plt.plot(history.history["accuracy"])
plt.xlabel("Epochs")
plt.ylabel("Accuracies")
plt.title("Simple RNN model")
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229076843-b075988e-3a4f-4194-9cb4-b0ff0e29b372.png">

    
<br>


## 6. train set 예측


```python
y_pred = regressor.predict(X_train)  # predictions
y_pred = scaler.inverse_transform(y_pred) # scaling back from 0-1 to original 예측값 -> 원래 값으로 변화
y_pred.shape
```

    37/37 [==============================] - 0s 12ms/step
    




    (1155, 1)




```python
y_train = scaler.inverse_transform(y_train) # scaling back from 0-1 to original
y_train.shape
```




    (1155, 1)



훈련한 모델을 가지고 예측한 y_pred와 실제값인 y_train를 비교한다.


```python
plt.figure(figsize = (30,10))
plt.plot(y_pred, color = "b", label = "y_pred" )
plt.plot(y_train, color = "g", label = "y_train")
plt.xlabel("Days")
plt.ylabel("Open price")
plt.title("Simple RNN model, Predictions with input X_train vs y_train")
plt.legend()
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229076918-69b4d220-7d50-47d5-825c-d0926746d7f6.png">
    

<br>

## 7. test set 예측


```python
# predictions with X_test data
y_pred_of_test = regressor.predict(X_test)
# scaling back from 0-1 to original
y_pred_of_test = scaler.inverse_transform(y_pred_of_test) 
print("Shape of y_pred_of_test :",y_pred_of_test.shape)
```

    15/15 [==============================] - 1s 12ms/step
    Shape of y_pred_of_test : (466, 1)
    


```python
# 예측값과 실제값의 차이
plt.figure(figsize = (30,10))
plt.plot(y_pred_of_test, label = "y_pred_of_test", c = "orange")
plt.plot(scaler.inverse_transform(y_test), label = "y_test", c = "g")
plt.xlabel("Days")
plt.ylabel("Open price")
plt.title("Simple RNN model, Prediction with input X_test vs y_test") # 예측값과 실제값의 차이
plt.legend()
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229077499-b296e8e7-e4ab-4188-8dca-4377e6a1911a.png">
    


앞서 예측한 train set의 시각화 test set의 시각화를 이어붙여준다. 역시나 모델이 학습된 것은 train set 기반이었기에에, train set을 예측한 앞부분은 오차가 적어보이는 반면, test set을 예측한 뒷부분은 오차가 꽤 있어보인다.

곤두박질 친 부분은 앞서 언급한 액면분할로 인한 거래중지의 data이다.


```python
plt.subplots(figsize =(30,12))
plt.plot(train_data.Date, train_data.Open, label = "train_data", color = "b")
plt.plot(validation_data.Date, validation_data.Open, label = "validation_data", color = "g")
plt.plot(train_data.Date.iloc[time_step:], y_pred, label = "y_pred", color = "r")
plt.plot(validation_data.Date.iloc[time_step:], y_pred_of_test, label = "y_pred_of_test", color = "orange")
plt.xlabel("Days")
plt.ylabel("Open price")
plt.title("Simple RNN model, Train-Validation-Prediction")
plt.legend()
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229076976-40930ba9-c272-4290-96e0-ad42750689cf.png">

    
<br>

## 8. Pytorch

이번엔 케라스가 아닌 파이토치로 모델을 구현해본다. 그리고 이전에 이전 Open 데이터로 미래 Open을 예측한 반면, 이번에는 Open, High, Low, Volume, Close로 미래 Open을 예측한다.


```python
import torch
import torch.nn as nn
import torch.optim as optim
```


```python
# 2016년~2022년 카카오 주가 데이터
data_pytorch = fdr.DataReader('035720', start='20160101', end='20221231')
```


```python
data_pytorch.reset_index(drop=False, inplace=True)
```


```python
scaler = MinMaxScaler()
data_pytorch[['Open','High','Low','Close','Volume']] = scaler.fit_transform(data_pytorch[['Open','High','Low','Close','Volume']])
data_pytorch.head()
```





  <div id="df-17322a8b-702c-4d77-83ba-32bd3c073528">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-01-04</td>
      <td>0.137110</td>
      <td>0.136318</td>
      <td>0.143739</td>
      <td>0.056905</td>
      <td>0.015755</td>
      <td>-0.004345</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-01-05</td>
      <td>0.133140</td>
      <td>0.136318</td>
      <td>0.141863</td>
      <td>0.059231</td>
      <td>0.016832</td>
      <td>0.015599</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-01-06</td>
      <td>0.137698</td>
      <td>0.140497</td>
      <td>0.146230</td>
      <td>0.061821</td>
      <td>0.028798</td>
      <td>0.017104</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-01-07</td>
      <td>0.138163</td>
      <td>0.139803</td>
      <td>0.145360</td>
      <td>0.059495</td>
      <td>0.018110</td>
      <td>-0.015101</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-01-08</td>
      <td>0.134547</td>
      <td>0.134462</td>
      <td>0.141739</td>
      <td>0.056776</td>
      <td>0.021172</td>
      <td>-0.017924</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-17322a8b-702c-4d77-83ba-32bd3c073528')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>

  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

      <script>
        const buttonEl =
          document.querySelector('#df-17322a8b-702c-4d77-83ba-32bd3c073528 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-17322a8b-702c-4d77-83ba-32bd3c073528');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
    </div>
  </div>





```python
data_pytorch.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1721 entries, 0 to 1720
    Data columns (total 7 columns):
     #   Column  Non-Null Count  Dtype         
    ---  ------  --------------  -----         
     0   Date    1721 non-null   datetime64[ns]
     1   Open    1721 non-null   float64       
     2   High    1721 non-null   float64       
     3   Low     1721 non-null   float64       
     4   Close   1721 non-null   float64       
     5   Volume  1721 non-null   float64       
     6   Change  1721 non-null   float64       
    dtypes: datetime64[ns](1), float64(6)
    memory usage: 94.2 KB
    

데이터셋을 feature와 target으로 나눈다.


```python
X = data_pytorch[['Open','High','Low','Volume', 'Close']].values
y = data_pytorch['Open'].values
```

앞서 했던 것처럼 time step을 50으로 잡고 데이터를 생성해준다.


```python
def seq_data(x, y, time_step):
  
  x_seq = []
  y_seq = []
  for i in range(len(x) - time_step):
    x_seq.append(x[i: i+time_step])
    y_seq.append(y[i+time_step])

  return torch.FloatTensor(x_seq), torch.FloatTensor(y_seq)
```


```python
time_step = 50

x_seq, y_seq = seq_data(X, y, time_step)
```

train과 test set을 7:3 비율로 split 해준다.


```python
length_data = x_seq.shape[0]    # data 행 개수
split_ratio = 0.7           # 0.7 / 0.3 으로 분리
length_train = round(length_data * split_ratio)

x_train_seq = x_seq[:length_train]
y_train_seq = y_seq[:length_train]
x_test_seq = x_seq[length_train:]
y_test_seq = y_seq[length_train:]
print(x_train_seq.size(), y_train_seq.size())
print(x_test_seq.size(), y_test_seq.size())
```

    torch.Size([1170, 50, 5]) torch.Size([1170])
    torch.Size([501, 50, 5]) torch.Size([501])
    

이제 min batch 형태로 쪼개질 수 있도록 `DataLoader()`를 사용하여 데이터를 iterator하게 만든다다.


```python
train = torch.utils.data.TensorDataset(x_train_seq, y_train_seq)
test = torch.utils.data.TensorDataset(x_test_seq, y_test_seq)

batch_size = 32
train_loader = torch.utils.data.DataLoader(dataset=train, batch_size=batch_size, shuffle=False)
test_loader = torch.utils.data.DataLoader(dataset=test, batch_size=batch_size, shuffle=False)
```

`input_size`는 feature의 개수이고, `num_layers`는 쌓고자 하는 층의 개수이다.


```python
input_dim = x_seq.size(2)
num_layers = 4
hidden_dim = 2
```

<br>

## 9. Pytorch RNN class 구성


```python
class My_RNN(nn.Module):
  def __init__(self, input_dim, hidden_dim, time_step, num_layers):
    super(My_RNN, self).__init__()
    self.hidden_dim = hidden_dim
    self.num_layers = num_layers
    self.rnn = nn.RNN(input_dim, hidden_dim, num_layers, dropout=0.2, batch_first=True)
    self.fc = nn.Sequential(nn.Linear(hidden_dim*time_step, 1), nn.Sigmoid())

  def forward(self, x):
    h0 = torch.zeros(self.num_layers, x.size()[0], self.hidden_dim)# 초기 hidden state 설정하기.
    out, _ = self.rnn(x, h0) # out: RNN의 마지막 레이어로부터 나온 output feature 를 반환한다. hn: hidden state를 반환한다.
    out = out.reshape(out.shape[0], -1) # many to many 전략
    out = self.fc(out)
    return out
```


```python
model = My_RNN(input_dim = input_dim,
                   hidden_dim = hidden_dim,
                   time_step = time_step,
                   num_layers = num_layers)
```


```python
criterion = nn.MSELoss()

lr = 1e-3
num_epochs = 50

loss_function = torch.nn.MSELoss() 
optimizer = optim.Adam(model.parameters(), lr=lr)
```


```python
loss_graph = [] # 그래프 그릴 목적인 loss.
n = len(train_loader)

for epoch in range(num_epochs):
  running_loss = 0.0

  for data in train_loader:

    seq, target = data # mini batch로 쪼개진 데이터터
    out = model(seq)   # model에서 forward
    loss = criterion(out, target) # output 기반으로로 loss 계산

    optimizer.zero_grad() # 
    loss.backward() # loss가 최소가 되게하는 
    optimizer.step() # 가중치 업데이트 해주고,
    running_loss += loss.item() # 한 mini batch의 loss 더해주고,

  loss_graph.append(running_loss / n) # 한 epoch에 모든 mini batch의 평균 loss 리스트에 담고,
  if epoch % 10 == 0:
    print('[epoch: %d] loss: %.4f'%(epoch, running_loss/n))
```

    [epoch: 0] loss: 0.0702
    [epoch: 10] loss: 0.0060
    [epoch: 20] loss: 0.0008
    [epoch: 30] loss: 0.0006
    [epoch: 40] loss: 0.0007
    


```python
plt.figure(figsize =(10,7))
plt.plot(loss_graph)
plt.xlabel("Epochs")
plt.ylabel("Losses")
plt.title("Simple RNN model")
plt.show()
```


    
<img src="https://user-images.githubusercontent.com/115082062/229077315-c7860ef1-f183-4663-a2d0-4fe51ac66557.png">
    

<br>

## 10. Pytorch 예측

이제 학습한 model로 train set과 test set을 예측한다. 아까처럼 두 그래프를 이어붙여서 시각화할 것이다.

결과를 보면 test set 부분이 많이 아쉽다. 이전에는 feature를 무엇을 쓸 것이고, layer를 어떻게 쌓느냐에 따라 결과가 많이 바뀔 것 같다.


```python
def plotting(train_loader, test_loader, actual):
  with torch.no_grad():
    train_pred = []
    test_pred = []

    for data in train_loader:
      seq, target = data
      out = model(seq)
      train_pred += out.numpy().tolist()

    for data in test_loader:
      seq, target = data
      out = model(seq)
      test_pred += out.numpy().tolist()
      
  total = train_pred + test_pred
  plt.figure(figsize=(20,10))
  plt.plot(np.ones(100)*len(train_pred), np.linspace(0,1,100), '--', linewidth=0.6)
  plt.plot(actual, '--')
  plt.plot(total, 'b', linewidth=0.6)

  plt.legend(['train boundary', 'actual', 'prediction'])
  plt.show()

plotting(train_loader, test_loader, data_pytorch['Open'][time_step:])
```


    
<img src="https://user-images.githubusercontent.com/115082062/229077234-698a079d-8471-47a2-8700-d7cf2dd2fa8e.png">
    

