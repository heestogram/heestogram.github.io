---
title: "[DL] RNN, LSTM, GRU를 활용한 미국 에너지 소비량 예측"
excerpt: "미국 전력 에너지 소비량 시계열 데이터를 PyTorch의 RNN, LSTM, GRU로 예측 및 비교해보자."
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, RNN, LSTM, GRU, 시계열, pytorch]

published: true

categories:
  - DL

date: 2023-02-15 21:00:00
last_modified_at: 2023-02-15 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

## 1. Installing Libaries



```python
import torch
import torch.nn as nn
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from datetime import datetime

device = "cuda" if torch.cuda.is_available() else "cpu"
print(f"{device}" " is available.")
```

    cpu is available.
    


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    

## 2. Loading Dataset, EDA

데이터셋은 미국 여러 지역의 전력 에너지 소비량을 시계열 데이터로 나타낸 것이다. 그 중 Duquesne(듀케인 in 펜실베니아)이라는 지역의 데이터셋을 택했다. 단위는 메가와트(MW)라고 한다.

관측기간은 2005년 1월 1일 ~ 2018년 8월 3일이다.



```python
df = pd.read_csv('/content/drive/MyDrive/kubig/DUQ_hourly.csv')
```


```python
df.head()
```





  <div id="df-0d0e852e-238c-4411-9951-103752472a03">
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
      <th>Datetime</th>
      <th>DUQ_MW</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005-12-31 01:00:00</td>
      <td>1458.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005-12-31 02:00:00</td>
      <td>1377.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005-12-31 03:00:00</td>
      <td>1351.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005-12-31 04:00:00</td>
      <td>1336.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005-12-31 05:00:00</td>
      <td>1356.0</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-0d0e852e-238c-4411-9951-103752472a03')"
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
          document.querySelector('#df-0d0e852e-238c-4411-9951-103752472a03 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-0d0e852e-238c-4411-9951-103752472a03');
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
df.describe()
```





  <div id="df-765f245f-2dc2-4fff-9902-37c55e3c167b">
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
      <th>DUQ_MW</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>119068.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1658.820296</td>
    </tr>
    <tr>
      <th>std</th>
      <td>301.740640</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1014.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1444.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1630.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1819.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3054.000000</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-765f245f-2dc2-4fff-9902-37c55e3c167b')"
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
          document.querySelector('#df-765f245f-2dc2-4fff-9902-37c55e3c167b button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-765f245f-2dc2-4fff-9902-37c55e3c167b');
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
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 119068 entries, 0 to 119067
    Data columns (total 2 columns):
     #   Column    Non-Null Count   Dtype  
    ---  ------    --------------   -----  
     0   Datetime  119068 non-null  object 
     1   DUQ_MW    119068 non-null  float64
    dtypes: float64(1), object(1)
    memory usage: 1.8+ MB
    


```python
import plotly.graph_objs as go
from plotly.offline import iplot

def plot_dataset(df, title):
    data = []
    
    value = go.Scatter(
        x=df.index,
        y=df.value,
        mode="lines",
        name="values",
        marker=dict(),
        text=df.index,
        line=dict(color="rgba(0,0,0, 0.3)"),
    )
    data.append(value)

    layout = dict(
        title=title,
        xaxis=dict(title="Date", ticklen=5, zeroline=False),
        yaxis=dict(title="Value", ticklen=5, zeroline=False),
    )

    fig = dict(data=data, layout=layout)
    iplot(fig)
    
```


```python
df = df.set_index(['Datetime'])
df = df.rename(columns={'DUQ_MW': 'value'})

df.index = pd.to_datetime(df.index)
if not df.index.is_monotonic:
    df = df.sort_index()
    
plot_dataset(df, title='Duquesne (DUQ) Region: estimated energy consumption in Megawatts (MW)')
```


<br>

## 2. Loading Dataset, EDA

데이터셋은 미국 여러 지역의 전력 에너지 소비량을 시계열 데이터로 나타낸 것이다. 그 중 Duquesne(듀케인 in 펜실베니아)이라는 지역의 데이터셋을 택했다. 단위는 메가와트(MW)라고 한다.

관측기간은 2005년 1월 1일 ~ 2018년 8월 3일이다.



```python
df = pd.read_csv('/content/drive/MyDrive/kubig/DUQ_hourly.csv')
```


```python
df.head()
```





  <div id="df-0d0e852e-238c-4411-9951-103752472a03">
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
      <th>Datetime</th>
      <th>DUQ_MW</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005-12-31 01:00:00</td>
      <td>1458.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005-12-31 02:00:00</td>
      <td>1377.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005-12-31 03:00:00</td>
      <td>1351.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005-12-31 04:00:00</td>
      <td>1336.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005-12-31 05:00:00</td>
      <td>1356.0</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-0d0e852e-238c-4411-9951-103752472a03')"
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
          document.querySelector('#df-0d0e852e-238c-4411-9951-103752472a03 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-0d0e852e-238c-4411-9951-103752472a03');
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
df.describe()
```





  <div id="df-765f245f-2dc2-4fff-9902-37c55e3c167b">
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
      <th>DUQ_MW</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>119068.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>1658.820296</td>
    </tr>
    <tr>
      <th>std</th>
      <td>301.740640</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1014.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1444.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1630.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1819.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3054.000000</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-765f245f-2dc2-4fff-9902-37c55e3c167b')"
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
          document.querySelector('#df-765f245f-2dc2-4fff-9902-37c55e3c167b button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-765f245f-2dc2-4fff-9902-37c55e3c167b');
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
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 119068 entries, 0 to 119067
    Data columns (total 2 columns):
     #   Column    Non-Null Count   Dtype  
    ---  ------    --------------   -----  
     0   Datetime  119068 non-null  object 
     1   DUQ_MW    119068 non-null  float64
    dtypes: float64(1), object(1)
    memory usage: 1.8+ MB
    


```python
import plotly.graph_objs as go
from plotly.offline import iplot

def plot_dataset(df, title):
    data = []
    
    value = go.Scatter(
        x=df.index,
        y=df.value,
        mode="lines",
        name="values",
        marker=dict(),
        text=df.index,
        line=dict(color="rgba(0,0,0, 0.3)"),
    )
    data.append(value)

    layout = dict(
        title=title,
        xaxis=dict(title="Date", ticklen=5, zeroline=False),
        yaxis=dict(title="Value", ticklen=5, zeroline=False),
    )

    fig = dict(data=data, layout=layout)
    iplot(fig)
    
```


```python
df = df.set_index(['Datetime'])
df = df.rename(columns={'DUQ_MW': 'value'})

df.index = pd.to_datetime(df.index)
if not df.index.is_monotonic:
    df = df.sort_index()
    
plot_dataset(df, title='Duquesne (DUQ) Region: estimated energy consumption in Megawatts (MW)')
```

<br>

## 3. Generating time-lagged observations, date/time features

본 데이터셋에서는 에너지 소비량을 제외하고는 다른 predictors가 전혀 없다. 때문에 feature를 새로이 생성할 필요가 있다. 시계열 예측에서 가장 보편적인 방식은 과거 데이터(lagged observation)를 생성해주는 것이다.

예를 들어 특정 시간의 value를 예측하기 위해 1시간 전 value, 2시간 전 value, ..., n시간 전 value를 새로운 컬럼으로 추가해주는 것이다. `shift()` 함수로 간단히 처리할 수 있다.

여기선 1~100시간 전 데이터를 lag1 ~ lag100으로 만들어 사용할 것이다.

하지만 이 경우 맨 처음 100개의 데이터는 소실된다는 단점이 존재한다. 그래서 원래는 1월 1일부터 시작했던 데이터가 첫 row가 1월 5일 05시로 바뀐 것을 확인할 수 있다.


```python
def generate_time_lags(df, n_lags):
    df_n = df.copy()
    for n in range(1, n_lags + 1):
        df_n[f"lag{n}"] = df_n["value"].shift(n)
    df_n = df_n.iloc[n_lags:]
    return df_n

input_dim = 100

df_timelags = generate_time_lags(df, input_dim)
df_timelags
```

    <ipython-input-6-9686bd7878a4>:4: PerformanceWarning:
    
    DataFrame is highly fragmented.  This is usually the result of calling `frame.insert` many times, which has poor performance.  Consider joining all columns at once using pd.concat(axis=1) instead.  To get a de-fragmented frame, use `newframe = frame.copy()`
    
    





  <div id="df-d21f4007-913b-4bb7-8249-3aa2cd60d0ad">
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
      <th>value</th>
      <th>lag1</th>
      <th>lag2</th>
      <th>lag3</th>
      <th>lag4</th>
      <th>lag5</th>
      <th>lag6</th>
      <th>lag7</th>
      <th>lag8</th>
      <th>lag9</th>
      <th>...</th>
      <th>lag91</th>
      <th>lag92</th>
      <th>lag93</th>
      <th>lag94</th>
      <th>lag95</th>
      <th>lag96</th>
      <th>lag97</th>
      <th>lag98</th>
      <th>lag99</th>
      <th>lag100</th>
    </tr>
    <tr>
      <th>Datetime</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>2005-01-05 05:00:00</th>
      <td>1369.0</td>
      <td>1372.0</td>
      <td>1344.0</td>
      <td>1364.0</td>
      <td>1442.0</td>
      <td>1486.0</td>
      <td>1608.0</td>
      <td>1735.0</td>
      <td>1771.0</td>
      <td>1818.0</td>
      <td>...</td>
      <td>1274.0</td>
      <td>1270.0</td>
      <td>1258.0</td>
      <td>1215.0</td>
      <td>1181.0</td>
      <td>1166.0</td>
      <td>1170.0</td>
      <td>1218.0</td>
      <td>1273.0</td>
      <td>1364.0</td>
    </tr>
    <tr>
      <th>2005-01-05 06:00:00</th>
      <td>1417.0</td>
      <td>1369.0</td>
      <td>1372.0</td>
      <td>1344.0</td>
      <td>1364.0</td>
      <td>1442.0</td>
      <td>1486.0</td>
      <td>1608.0</td>
      <td>1735.0</td>
      <td>1771.0</td>
      <td>...</td>
      <td>1330.0</td>
      <td>1274.0</td>
      <td>1270.0</td>
      <td>1258.0</td>
      <td>1215.0</td>
      <td>1181.0</td>
      <td>1166.0</td>
      <td>1170.0</td>
      <td>1218.0</td>
      <td>1273.0</td>
    </tr>
    <tr>
      <th>2005-01-05 07:00:00</th>
      <td>1569.0</td>
      <td>1417.0</td>
      <td>1369.0</td>
      <td>1372.0</td>
      <td>1344.0</td>
      <td>1364.0</td>
      <td>1442.0</td>
      <td>1486.0</td>
      <td>1608.0</td>
      <td>1735.0</td>
      <td>...</td>
      <td>1352.0</td>
      <td>1330.0</td>
      <td>1274.0</td>
      <td>1270.0</td>
      <td>1258.0</td>
      <td>1215.0</td>
      <td>1181.0</td>
      <td>1166.0</td>
      <td>1170.0</td>
      <td>1218.0</td>
    </tr>
    <tr>
      <th>2005-01-05 08:00:00</th>
      <td>1736.0</td>
      <td>1569.0</td>
      <td>1417.0</td>
      <td>1369.0</td>
      <td>1372.0</td>
      <td>1344.0</td>
      <td>1364.0</td>
      <td>1442.0</td>
      <td>1486.0</td>
      <td>1608.0</td>
      <td>...</td>
      <td>1371.0</td>
      <td>1352.0</td>
      <td>1330.0</td>
      <td>1274.0</td>
      <td>1270.0</td>
      <td>1258.0</td>
      <td>1215.0</td>
      <td>1181.0</td>
      <td>1166.0</td>
      <td>1170.0</td>
    </tr>
    <tr>
      <th>2005-01-05 09:00:00</th>
      <td>1809.0</td>
      <td>1736.0</td>
      <td>1569.0</td>
      <td>1417.0</td>
      <td>1369.0</td>
      <td>1372.0</td>
      <td>1344.0</td>
      <td>1364.0</td>
      <td>1442.0</td>
      <td>1486.0</td>
      <td>...</td>
      <td>1356.0</td>
      <td>1371.0</td>
      <td>1352.0</td>
      <td>1330.0</td>
      <td>1274.0</td>
      <td>1270.0</td>
      <td>1258.0</td>
      <td>1215.0</td>
      <td>1181.0</td>
      <td>1166.0</td>
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
      <th>2018-08-02 20:00:00</th>
      <td>1966.0</td>
      <td>1999.0</td>
      <td>2050.0</td>
      <td>2039.0</td>
      <td>2029.0</td>
      <td>1983.0</td>
      <td>1952.0</td>
      <td>1931.0</td>
      <td>1865.0</td>
      <td>1792.0</td>
      <td>...</td>
      <td>1453.0</td>
      <td>1557.0</td>
      <td>1675.0</td>
      <td>1789.0</td>
      <td>1817.0</td>
      <td>1866.0</td>
      <td>1907.0</td>
      <td>1943.0</td>
      <td>1922.0</td>
      <td>1862.0</td>
    </tr>
    <tr>
      <th>2018-08-02 21:00:00</th>
      <td>1944.0</td>
      <td>1966.0</td>
      <td>1999.0</td>
      <td>2050.0</td>
      <td>2039.0</td>
      <td>2029.0</td>
      <td>1983.0</td>
      <td>1952.0</td>
      <td>1931.0</td>
      <td>1865.0</td>
      <td>...</td>
      <td>1376.0</td>
      <td>1453.0</td>
      <td>1557.0</td>
      <td>1675.0</td>
      <td>1789.0</td>
      <td>1817.0</td>
      <td>1866.0</td>
      <td>1907.0</td>
      <td>1943.0</td>
      <td>1922.0</td>
    </tr>
    <tr>
      <th>2018-08-02 22:00:00</th>
      <td>1901.0</td>
      <td>1944.0</td>
      <td>1966.0</td>
      <td>1999.0</td>
      <td>2050.0</td>
      <td>2039.0</td>
      <td>2029.0</td>
      <td>1983.0</td>
      <td>1952.0</td>
      <td>1931.0</td>
      <td>...</td>
      <td>1332.0</td>
      <td>1376.0</td>
      <td>1453.0</td>
      <td>1557.0</td>
      <td>1675.0</td>
      <td>1789.0</td>
      <td>1817.0</td>
      <td>1866.0</td>
      <td>1907.0</td>
      <td>1943.0</td>
    </tr>
    <tr>
      <th>2018-08-02 23:00:00</th>
      <td>1789.0</td>
      <td>1901.0</td>
      <td>1944.0</td>
      <td>1966.0</td>
      <td>1999.0</td>
      <td>2050.0</td>
      <td>2039.0</td>
      <td>2029.0</td>
      <td>1983.0</td>
      <td>1952.0</td>
      <td>...</td>
      <td>1312.0</td>
      <td>1332.0</td>
      <td>1376.0</td>
      <td>1453.0</td>
      <td>1557.0</td>
      <td>1675.0</td>
      <td>1789.0</td>
      <td>1817.0</td>
      <td>1866.0</td>
      <td>1907.0</td>
    </tr>
    <tr>
      <th>2018-08-03 00:00:00</th>
      <td>1656.0</td>
      <td>1789.0</td>
      <td>1901.0</td>
      <td>1944.0</td>
      <td>1966.0</td>
      <td>1999.0</td>
      <td>2050.0</td>
      <td>2039.0</td>
      <td>2029.0</td>
      <td>1983.0</td>
      <td>...</td>
      <td>1330.0</td>
      <td>1312.0</td>
      <td>1332.0</td>
      <td>1376.0</td>
      <td>1453.0</td>
      <td>1557.0</td>
      <td>1675.0</td>
      <td>1789.0</td>
      <td>1817.0</td>
      <td>1866.0</td>
    </tr>
  </tbody>
</table>
<p>118968 rows × 101 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-d21f4007-913b-4bb7-8249-3aa2cd60d0ad')"
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
          document.querySelector('#df-d21f4007-913b-4bb7-8249-3aa2cd60d0ad button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-d21f4007-913b-4bb7-8249-3aa2cd60d0ad');
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




또한 인덱스로 지정한 `Datetime을 이용하여 시간, 날짜, 월, 요일, 한 해 중 몇 주차 등의 feature를 생성할 수 있다.



```python
df = df.reset_index(drop=False)
df['HOUR'] = df['Datetime'].dt.hour #시간 변수 추가
df['MONTH'] = df['Datetime'].dt.month #월 변수 추가
df['WEEK'] = df['Datetime'].dt.weekofyear #주차 변수 추가
df['DAY'] = df['Datetime'].dt.dayofyear #일 변수 추가
df['WEEKDAY'] = df['Datetime'].dt.weekday #요일 변수 추가
```

    <ipython-input-7-29a1151e7af4>:4: FutureWarning:
    
    Series.dt.weekofyear and Series.dt.week have been deprecated.  Please use Series.dt.isocalendar().week instead.
    
    

그리고 각 datetime features로 groupby를 해서 그래프 추이를 확인해보았다.


```python
draw_list = ["HOUR","DAY","MONTH","WEEK","WEEKDAY"]

#시간 변수를 x축에, 전력 소비량을 y축에 두고 그린 line plot 함수
def plot_per_time(sets, draw_list):
    plt.figure(figsize=(20,10))
    idx=1
    for var in draw_list:
        groupby = sets.groupby(var).mean()
        groupby.reset_index(inplace=True)
        plt.subplot(3,2,idx)
        plt.plot(groupby[var], groupby['value'])
        plt.title(var)
        idx+=1
        
plot_per_time(df, draw_list)
```


    
<img src="https://user-images.githubusercontent.com/115082062/229084143-7cfcac13-af48-4537-a995-9bcff55926b6.png">

<br>
    


- HOUR: 주로 새벽 시간대에는 소비량이 낮거나 거의 없다. 하루 중 가장 높은 시간대는 17~18시경으로 보이며 이는 밤이 찾아옴에 따라 전력량이 높아지고, 직장인들의 퇴근 직전, 상권이 활발해지기 시작하는 시간, 가정집에서도 전력이 소비되기 시작하는 시간대라 그런 것으로 사료된다.
- DAY, MONTH, WEEK: 기후가 온화한 봄, 가을에는 소비량이 낮고 여름에 가장 높고 겨울이 그를 뒤따른다. 냉난방 장치의 가동으로 이유를 쉽게 추론할 수 있다. 
- WEEKDAY: 주말(5,6)에는 직장인들이 출근하지 않는 경우가 일반적이니 소비량이 적어든 것으로 사료된다.

<br>

## 4. One-hot encoding

datetime features들은 값이 크다고 하여 영향력이 큰 것이 아니라 그저 범주형 변수이므로 값의 크기가 예측에 영향을 주면 안된다다. 때문에 원핫인코딩을 해줄 필요가 있다. pandas에선 `get_dummies`를 사용할 수 있다.

DAY를 원핫인코딩을 하면 365개의 feature가 추가되어 모델의 복잡도를 높일 수 있으니 MONTH, WEEK, WEEKDAY에 대해서만 원핫인코딩을 한다.


```python
def onehot_encode_pd(df, cols):
    for col in cols:
        dummies = pd.get_dummies(df[col], prefix=col)
        df = pd.concat([df, dummies], axis=1)
    
    return df.drop(columns=cols)

df_features = onehot_encode_pd(df, ['MONTH','WEEK','WEEKDAY'])
df_features
```





  <div id="df-f1baa507-175b-4232-a37d-276968324b52">
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
      <th>Datetime</th>
      <th>value</th>
      <th>HOUR</th>
      <th>DAY</th>
      <th>MONTH_1</th>
      <th>MONTH_2</th>
      <th>MONTH_3</th>
      <th>MONTH_4</th>
      <th>MONTH_5</th>
      <th>MONTH_6</th>
      <th>...</th>
      <th>WEEK_51</th>
      <th>WEEK_52</th>
      <th>WEEK_53</th>
      <th>WEEKDAY_0</th>
      <th>WEEKDAY_1</th>
      <th>WEEKDAY_2</th>
      <th>WEEKDAY_3</th>
      <th>WEEKDAY_4</th>
      <th>WEEKDAY_5</th>
      <th>WEEKDAY_6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005-01-01 01:00:00</td>
      <td>1364.0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005-01-01 02:00:00</td>
      <td>1273.0</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005-01-01 03:00:00</td>
      <td>1218.0</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005-01-01 04:00:00</td>
      <td>1170.0</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005-01-01 05:00:00</td>
      <td>1166.0</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
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
      <th>119063</th>
      <td>2018-08-02 20:00:00</td>
      <td>1966.0</td>
      <td>20</td>
      <td>214</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>119064</th>
      <td>2018-08-02 21:00:00</td>
      <td>1944.0</td>
      <td>21</td>
      <td>214</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>119065</th>
      <td>2018-08-02 22:00:00</td>
      <td>1901.0</td>
      <td>22</td>
      <td>214</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>119066</th>
      <td>2018-08-02 23:00:00</td>
      <td>1789.0</td>
      <td>23</td>
      <td>214</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>119067</th>
      <td>2018-08-03 00:00:00</td>
      <td>1656.0</td>
      <td>0</td>
      <td>215</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>119068 rows × 76 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-f1baa507-175b-4232-a37d-276968324b52')"
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
          document.querySelector('#df-f1baa507-175b-4232-a37d-276968324b52 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-f1baa507-175b-4232-a37d-276968324b52');
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


<br>

## 5. Generating cyclical features (sin/cos transformations)

시간은 0~23이라는 정수형 타입으로 표현된다. 그런데 이렇게되면 1월 1일 23시와 1월 2일 00시가 실제로 한 시간의 차이를 가짐에도 23의 차리을 갖는 꼴이 된다. 그래서 이를 해결하고자 삼각함수로 변환하여 시간이 연속성을 갖게 할 것이다.

HOUR, DAY에 대해서만 삼각변수 변환을 해주고 나머지는 그대로 둔다.


```python
def generate_cyclical_features(df, col_name, period, start_num=0):
    kwargs = {
        f'sin_{col_name}' : lambda x: np.sin(2*np.pi*(df[col_name]-start_num)/period),
        f'cos_{col_name}' : lambda x: np.cos(2*np.pi*(df[col_name]-start_num)/period)    
             }
    return df.assign(**kwargs).drop(columns=[col_name])

df_features = generate_cyclical_features(df_features, 'HOUR', 24, 0)
df_features = generate_cyclical_features(df_features, 'DAY', 365, 1)
# df_features = generate_cyclical_features(df_features, 'day_of_week', 7, 0)
# df_features = generate_cyclical_features(df_features, 'month', 12, 1)
# df_features = generate_cyclical_features(df_features, 'week_of_year', 52, 0)

df_features
```





  <div id="df-c3aa9151-0548-40db-8e5a-76d257e62a58">
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
      <th>Datetime</th>
      <th>value</th>
      <th>MONTH_1</th>
      <th>MONTH_2</th>
      <th>MONTH_3</th>
      <th>MONTH_4</th>
      <th>MONTH_5</th>
      <th>MONTH_6</th>
      <th>MONTH_7</th>
      <th>MONTH_8</th>
      <th>...</th>
      <th>WEEKDAY_1</th>
      <th>WEEKDAY_2</th>
      <th>WEEKDAY_3</th>
      <th>WEEKDAY_4</th>
      <th>WEEKDAY_5</th>
      <th>WEEKDAY_6</th>
      <th>sin_HOUR</th>
      <th>cos_HOUR</th>
      <th>sin_DAY</th>
      <th>cos_DAY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005-01-01 01:00:00</td>
      <td>1364.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.258819</td>
      <td>0.965926</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005-01-01 02:00:00</td>
      <td>1273.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.500000</td>
      <td>0.866025</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005-01-01 03:00:00</td>
      <td>1218.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.707107</td>
      <td>0.707107</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005-01-01 04:00:00</td>
      <td>1170.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.866025</td>
      <td>0.500000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005-01-01 05:00:00</td>
      <td>1166.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.965926</td>
      <td>0.258819</td>
      <td>0.000000</td>
      <td>1.000000</td>
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
      <th>119063</th>
      <td>2018-08-02 20:00:00</td>
      <td>1966.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.866025</td>
      <td>0.500000</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
    </tr>
    <tr>
      <th>119064</th>
      <td>2018-08-02 21:00:00</td>
      <td>1944.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.707107</td>
      <td>0.707107</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
    </tr>
    <tr>
      <th>119065</th>
      <td>2018-08-02 22:00:00</td>
      <td>1901.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.500000</td>
      <td>0.866025</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
    </tr>
    <tr>
      <th>119066</th>
      <td>2018-08-02 23:00:00</td>
      <td>1789.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.258819</td>
      <td>0.965926</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
    </tr>
    <tr>
      <th>119067</th>
      <td>2018-08-03 00:00:00</td>
      <td>1656.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-0.516062</td>
      <td>-0.856551</td>
    </tr>
  </tbody>
</table>
<p>119068 rows × 78 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-c3aa9151-0548-40db-8e5a-76d257e62a58')"
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
          document.querySelector('#df-c3aa9151-0548-40db-8e5a-76d257e62a58 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-c3aa9151-0548-40db-8e5a-76d257e62a58');
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


<br>

## 6. Generating Holiday features

앞서 그래프로 확인하였듯 주말 여부에 따라 전력량이 큰 폭으로 바뀌고 있다. 따라서 주말 여부를 binary하게 나타내준다.



```python
from datetime import date
import holidays

df_features.set_index(['Datetime'], inplace=True)
us_holidays = holidays.US()

def is_holiday(date):
    date = date.replace(hour = 0)
    return 1 if (date in us_holidays) else 0

def add_holiday_col(df, holidays):
    return df.assign(is_holiday = df.index.to_series().apply(is_holiday))


df_features = add_holiday_col(df_features, us_holidays)
df_features
```





  <div id="df-920b9f75-cafc-468f-84fc-d40942a56e08">
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
      <th>value</th>
      <th>MONTH_1</th>
      <th>MONTH_2</th>
      <th>MONTH_3</th>
      <th>MONTH_4</th>
      <th>MONTH_5</th>
      <th>MONTH_6</th>
      <th>MONTH_7</th>
      <th>MONTH_8</th>
      <th>MONTH_9</th>
      <th>...</th>
      <th>WEEKDAY_2</th>
      <th>WEEKDAY_3</th>
      <th>WEEKDAY_4</th>
      <th>WEEKDAY_5</th>
      <th>WEEKDAY_6</th>
      <th>sin_HOUR</th>
      <th>cos_HOUR</th>
      <th>sin_DAY</th>
      <th>cos_DAY</th>
      <th>is_holiday</th>
    </tr>
    <tr>
      <th>Datetime</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>2005-01-01 01:00:00</th>
      <td>1364.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.258819</td>
      <td>0.965926</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2005-01-01 02:00:00</th>
      <td>1273.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.500000</td>
      <td>0.866025</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2005-01-01 03:00:00</th>
      <td>1218.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.707107</td>
      <td>0.707107</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2005-01-01 04:00:00</th>
      <td>1170.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.866025</td>
      <td>0.500000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2005-01-01 05:00:00</th>
      <td>1166.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.965926</td>
      <td>0.258819</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1</td>
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
      <th>2018-08-02 20:00:00</th>
      <td>1966.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.866025</td>
      <td>0.500000</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2018-08-02 21:00:00</th>
      <td>1944.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.707107</td>
      <td>0.707107</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2018-08-02 22:00:00</th>
      <td>1901.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.500000</td>
      <td>0.866025</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2018-08-02 23:00:00</th>
      <td>1789.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.258819</td>
      <td>0.965926</td>
      <td>-0.501242</td>
      <td>-0.865307</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2018-08-03 00:00:00</th>
      <td>1656.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-0.516062</td>
      <td>-0.856551</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>119068 rows × 78 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-920b9f75-cafc-468f-84fc-d40942a56e08')"
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
          document.querySelector('#df-920b9f75-cafc-468f-84fc-d40942a56e08 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-920b9f75-cafc-468f-84fc-d40942a56e08');
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


<br>

## 7. Splitting the data into test, validation and train sets


```python
from sklearn.model_selection import train_test_split

def feature_label_split(df, target_col):
    y = df[[target_col]]
    X = df.drop(columns=[target_col])
    return X, y

def train_val_test_split(df, target_col, test_ratio):
    val_ratio = test_ratio / (1 - test_ratio)
    X, y = feature_label_split(df, target_col)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=test_ratio, shuffle=False)
    X_train, X_val, y_train, y_val = train_test_split(X_train, y_train, test_size=val_ratio, shuffle=False)
    return X_train, X_val, X_test, y_train, y_val, y_test

X_train, X_val, X_test, y_train, y_val, y_test = train_val_test_split(df_features, 'value', 0.2)
```


```python
print(X_train.shape)
print(X_val.shape)
print(X_test.shape)
```

    (71440, 77)
    (23814, 77)
    (23814, 77)
    
<br>

## 8. Scaling

스케일링은 가중치 업데이트를 훨씬 용이하게 해주므로 신경망에서 효과적이다.



```python
from sklearn.preprocessing import MinMaxScaler, StandardScaler, MaxAbsScaler, RobustScaler

def get_scaler(scaler):
    scalers = {
        "minmax": MinMaxScaler,
        "standard": StandardScaler,
        "maxabs": MaxAbsScaler,
        "robust": RobustScaler,
    }
    return scalers.get(scaler.lower())()
```


```python
scaler = get_scaler('minmax')
X_train_arr = scaler.fit_transform(X_train)
X_val_arr = scaler.transform(X_val)
X_test_arr = scaler.transform(X_test)

y_train_arr = scaler.fit_transform(y_train)
y_val_arr = scaler.transform(y_val)
y_test_arr = scaler.transform(y_test)
```

<br>

## 9. Loading the data into DataLoaders

PyTorch의 `DataLoader` class를 사용하면 신경망 학습에 필요한 mini-batch 형태로 쪼개어 데이터를 iterable하게 만들어준다.

그에 앞서 데이터의 형태를 Tensor형으로 바꾸어준다.



```python
from torch.utils.data import TensorDataset, DataLoader

batch_size = 64

train_features = torch.Tensor(X_train_arr)
train_targets = torch.Tensor(y_train_arr)
val_features = torch.Tensor(X_val_arr)
val_targets = torch.Tensor(y_val_arr)
test_features = torch.Tensor(X_test_arr)
test_targets = torch.Tensor(y_test_arr)

train = TensorDataset(train_features, train_targets)
val = TensorDataset(val_features, val_targets)
test = TensorDataset(test_features, test_targets)

train_loader = DataLoader(train, batch_size=batch_size, shuffle=False, drop_last=True)
val_loader = DataLoader(val, batch_size=batch_size, shuffle=False, drop_last=True)
test_loader = DataLoader(test, batch_size=batch_size, shuffle=False, drop_last=True)
test_loader_one = DataLoader(test, batch_size=1, shuffle=False, drop_last=True)
```

<br>

## 10. Defining the RNN model classes(RNN, LSTM, GRU)


### 10-1. RNN



```python
class RNNModel(nn.Module):
    def __init__(self, input_dim, hidden_dim, layer_dim, output_dim, dropout_prob):
        """The __init__ method that initiates an RNN instance.

        Args:
            input_dim (int): The number of nodes in the input layer
            hidden_dim (int): The number of nodes in each layer
            layer_dim (int): The number of layers in the network
            output_dim (int): The number of nodes in the output layer
            dropout_prob (float): The probability of nodes being dropped out

        """
        # __init__ 메소드에서 super(RNNModel, self).__init_()을 해줌으로써 
        # 호출 단계에서 부모 클래스(nn.Module)의 __init__ 메소드를 호출해주고, 다양한 변수들을 상속받을 수 있음.
        super(RNNModel, self).__init__()

        # 각 층을 몇 개 쌓을 것인지 정의
        self.hidden_dim = hidden_dim
        self.layer_dim = layer_dim

        # RNN layers
        self.rnn = nn.RNN(
            input_dim, hidden_dim, layer_dim, batch_first=True, dropout=dropout_prob
        )
        # 모든 뉴런이 다음 층의 모든 뉴런과 연결되는 '완전연결층'
        # 평탄화(Flatten)를 한다.
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        """forward는 입력값인 tensor를 forward propadation(순전파)한다

        인자:
            x (torch.Tensor): 입력 tensor의 차원은 (batch size, sequence length, input_dim)

        결과값:
            torch.Tensor: The output tensor of the shape (batch size, output_dim)

        """
        # 초기 은닉상태를 0으로 초기화해준다.
        h0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()

        # 순전파 과정(input -> hidden)
        out, h0 = self.rnn(x, h0.detach())

        # 3차원(batch_size, seq_length, hidden_size)으로 구성된 output을 마지막 seq_lenth만 갖고옴으로써 2차원으로
        # 마지막 출력만 가지고 FC층에 넣을 준비하는 것임
        out = out[:, -1, :]

        # Convert the final state to our desired output shape (batch_size, output_dim)
        out = self.fc(out)
        return out
```

<br>

### 10-2. LSTM
출력, 입력, 삭제 게이트를 갖고 있어 불필요한 기억은 삭제하고 장기 기억에 특화된 모델이다.


```python
class LSTMModel(nn.Module):
    """LSTMModel class extends nn.Module class and works as a constructor for LSTMs.

       LSTMModel class initiates a LSTM module based on PyTorch's nn.Module class.
       It has only two methods, namely init() and forward(). While the init()
       method initiates the model with the given input parameters, the forward()
       method defines how the forward propagation needs to be calculated.
       Since PyTorch automatically defines back propagation, there is no need
       to define back propagation method.

       Attributes:
           hidden_dim (int): The number of nodes in each layer
           layer_dim (str): The number of layers in the network
           lstm (nn.LSTM): The LSTM model constructed with the input parameters.
           fc (nn.Linear): The fully connected layer to convert the final state
                           of LSTMs to our desired output shape.

    """
    def __init__(self, input_dim, hidden_dim, layer_dim, output_dim, dropout_prob):
        """The __init__ method that initiates a LSTM instance.

        Args:
            input_dim (int): The number of nodes in the input layer
            hidden_dim (int): The number of nodes in each layer
            layer_dim (int): The number of layers in the network
            output_dim (int): The number of nodes in the output layer
            dropout_prob (float): The probability of nodes being dropped out

        """
        super(LSTMModel, self).__init__()

        # Defining the number of layers and the nodes in each layer
        self.hidden_dim = hidden_dim
        self.layer_dim = layer_dim

        # LSTM layers
        self.lstm = nn.LSTM(
            input_dim, hidden_dim, layer_dim, batch_first=True, dropout=dropout_prob
        )

        # Fully connected layer
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        """The forward method takes input tensor x and does forward propagation

        Args:
            x (torch.Tensor): The input tensor of the shape (batch size, sequence length, input_dim)

        Returns:
            torch.Tensor: The output tensor of the shape (batch size, output_dim)

        """
        # Initializing hidden state for first input with zeros
        h0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()

        # RNN과 달리 LSTM에는 cell state(셀 상태)가 존재한다. 이 역시 hidden state처럼 0으로 생성해준다.
        c0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()

        # We need to detach as we are doing truncated backpropagation through time (BPTT)
        # If we don't, we'll backprop all the way to the start even after going through another batch
        # Forward propagation by passing in the input, hidden state, and cell state into the model
        out, (hn, cn) = self.lstm(x, (h0.detach(), c0.detach()))

        # Reshaping the outputs in the shape of (batch_size, seq_length, hidden_size)
        # so that it can fit into the fully connected layer
        out = out[:, -1, :]

        # Convert the final state to our desired output shape (batch_size, output_dim)
        out = self.fc(out)

        return out
```

<br>

### 10-3 GRU
LSTM처럼 장기 기억에 효과적이지만 GRU는 업데이트 게이트와 리셋 게이트만 존재한다. 또한 cell state(셀 상태)도 존재하지 않는다.



```python
class GRUModel(nn.Module):
    """GRUModel class extends nn.Module class and works as a constructor for GRUs.

       GRUModel class initiates a GRU module based on PyTorch's nn.Module class.
       It has only two methods, namely init() and forward(). While the init()
       method initiates the model with the given input parameters, the forward()
       method defines how the forward propagation needs to be calculated.
       Since PyTorch automatically defines back propagation, there is no need
       to define back propagation method.

       Attributes:
           hidden_dim (int): The number of nodes in each layer
           layer_dim (str): The number of layers in the network
           gru (nn.GRU): The GRU model constructed with the input parameters.
           fc (nn.Linear): The fully connected layer to convert the final state
                           of GRUs to our desired output shape.

    """
    def __init__(self, input_dim, hidden_dim, layer_dim, output_dim, dropout_prob):
        """The __init__ method that initiates a GRU instance.

        Args:
            input_dim (int): The number of nodes in the input layer
            hidden_dim (int): The number of nodes in each layer
            layer_dim (int): The number of layers in the network
            output_dim (int): The number of nodes in the output layer
            dropout_prob (float): The probability of nodes being dropped out

        """
        super(GRUModel, self).__init__()

        # Defining the number of layers and the nodes in each layer
        self.layer_dim = layer_dim
        self.hidden_dim = hidden_dim

        # GRU layers
        self.gru = nn.GRU(
            input_dim, hidden_dim, layer_dim, batch_first=True, dropout=dropout_prob
        )

        # Fully connected layer
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        """The forward method takes input tensor x and does forward propagation

        Args:
            x (torch.Tensor): The input tensor of the shape (batch size, sequence length, input_dim)

        Returns:
            torch.Tensor: The output tensor of the shape (batch size, output_dim)

        """
        # Initializing hidden state for first input with zeros
        h0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()

        # Forward propagation by passing in the input and hidden state into the model
        # 
        out, _ = self.gru(x, h0.detach())

        # Reshaping the outputs in the shape of (batch_size, seq_length, hidden_size)
        # so that it can fit into the fully connected layer
        out = out[:, -1, :]

        # Convert the final state to our desired output shape (batch_size, output_dim)
        out = self.fc(out)

        return out

```


```python
def get_model(model, model_params):
    models = {
        "rnn": RNNModel,
        "lstm": LSTMModel,
        "gru": GRUModel,
    }
    return models.get(model.lower())(**model_params)
```

<br>

## 11. Making Predictions
아래에선 모델로 훈련 데이터를 예측하고 오차를 계산하고 옵티마이징을 하는 과정을 `train_step()`메서드로 만들어주었다. 이것이 하나의 epoch이 된다. 그리고 `train()` 메서드에서는 각각의 epoch을 반복해주는 역할을 한다. 그 밖의 validation, evaluate를 할 수 있는 메서드를 종합하여 `optimization`라는 class를 만들었다.

train과 evaluation에서 가장 큰 차이점은 전자는 가중치를 업데이트하지만, 후자는 그렇지 않다는 점이다.


```python
class Optimization:
    """Optimization is a helper class that allows training, validation, prediction.

    Optimization is a helper class that takes model, loss function, optimizer function
    learning scheduler (optional), early stopping (optional) as inputs. In return, it
    provides a framework to train and validate the models, and to predict future values
    based on the models.

    Attributes:
        model (RNNModel, LSTMModel, GRUModel): Model class created for the type of RNN
        loss_fn (torch.nn.modules.Loss): Loss function to calculate the losses
        optimizer (torch.optim.Optimizer): Optimizer function to optimize the loss function
        train_losses (list[float]): The loss values from the training
        val_losses (list[float]): The loss values from the validation
        last_epoch (int): The number of epochs that the models is trained
    """
    def __init__(self, model, loss_fn, optimizer):
        """
        Args:
            model (RNNModel, LSTMModel, GRUModel): Model class created for the type of RNN
            loss_fn (torch.nn.modules.Loss): Loss function to calculate the losses
            optimizer (torch.optim.Optimizer): Optimizer function to optimize the loss function
        """
        self.model = model
        self.loss_fn = loss_fn
        self.optimizer = optimizer
        self.train_losses = []
        self.val_losses = []
        
    # training의 one step(=one epoch)을 담당하는 함수
    def train_step(self, x, y):
        """The method train_step completes one step of training.

        Given the features (x) and the target values (y) tensors, the method completes
        one step of the training. First, it activates the train mode to enable back prop.
        After generating predicted values (yhat) by doing forward propagation, it calculates
        the losses by using the loss function. Then, it computes the gradients by doing
        back propagation and updates the weights by calling step() function.

        Args:
            x (torch.Tensor): Tensor for features to train one step
            y (torch.Tensor): Tensor for target values to calculate losses

        """
        # Sets model to train mode
        self.model.train()

        # Makes predictions
        yhat = self.model(x)

        # Computes loss
        loss = self.loss_fn(y, yhat)

        # Computes gradients
        loss.backward()

        # Updates parameters and zeroes gradients
        self.optimizer.step()
        self.optimizer.zero_grad()

        # Returns the loss
        return loss.item()

    def train(self, train_loader, val_loader, batch_size=64, n_epochs=50, n_features=1):
        """The method train performs the model training

        The method takes DataLoaders for training and validation datasets, batch size for
        mini-batch training, number of epochs to train, and number of features as inputs.
        Then, it carries out the training by iteratively calling the method train_step for
        n_epochs times. If early stopping is enabled, then it  checks the stopping condition
        to decide whether the training needs to halt before n_epochs steps. Finally, it saves
        the model in a designated file path.

        Args:
            train_loader (torch.utils.data.DataLoader): DataLoader that stores training data
            val_loader (torch.utils.data.DataLoader): DataLoader that stores validation data
            batch_size (int): Batch size for mini-batch training
            n_epochs (int): Number of epochs, i.e., train steps, to train
            n_features (int): Number of feature columns

        """
        model_path = f'{self.model}_{datetime.now().strftime("%Y-%m-%d %H:%M:%S")}'

        # train_step을 n_epoch만큼 반복해준다.
        for epoch in range(1, n_epochs + 1):
            batch_losses = []
            # mini-batch로 쪼개진 단위씩 학습
            for x_batch, y_batch in train_loader:
                x_batch = x_batch.view([batch_size, -1, n_features]).to(device)
                y_batch = y_batch.to(device)
                loss = self.train_step(x_batch, y_batch)
                batch_losses.append(loss)
            training_loss = np.mean(batch_losses)
            self.train_losses.append(training_loss)

            # validation을 할 때에는 가중치 업데이트를 비활성화한다.
            with torch.no_grad():
                batch_val_losses = []
                for x_val, y_val in val_loader:
                    x_val = x_val.view([batch_size, -1, n_features]).to(device)
                    y_val = y_val.to(device)
                    self.model.eval()
                    yhat = self.model(x_val)
                    val_loss = self.loss_fn(y_val, yhat).item()
                    batch_val_losses.append(val_loss)
                validation_loss = np.mean(batch_val_losses)
                self.val_losses.append(validation_loss)

            if (epoch <= 10) | (epoch % 50 == 0):
                print(
                    f"[{epoch}/{n_epochs}] Training loss: {training_loss:.4f}\t Validation loss: {validation_loss:.4f}"
                )

        torch.save(self.model.state_dict(), model_path)

    def evaluate(self, test_loader, batch_size=1, n_features=1):
        """The method evaluate performs the model evaluation

        The method takes DataLoaders for the test dataset, batch size for mini-batch testing,
        and number of features as inputs. Similar to the model validation, it iteratively
        predicts the target values and calculates losses. Then, it returns two lists that
        hold the predictions and the actual values.

        Note:
            This method assumes that the prediction from the previous step is available at
            the time of the prediction, and only does one-step prediction into the future.

        Args:
            test_loader (torch.utils.data.DataLoader): DataLoader that stores test data
            batch_size (int): Batch size for mini-batch training
            n_features (int): Number of feature columns

        Returns:
            list[float]: The values predicted by the model
            list[float]: The actual values in the test set.

        """
        # evaluation을 할 때에는 가중치 업데이트를 비활성화한다.
        with torch.no_grad():
            predictions = []
            values = []
            for x_test, y_test in test_loader:
                x_test = x_test.view([batch_size, -1, n_features]).to(device)
                y_test = y_test.to(device)
                self.model.eval()
                yhat = self.model(x_test)
                predictions.append(yhat.to(device).detach().numpy())
                values.append(y_test.to(device).detach().numpy())

        return predictions, values

    def plot_losses(self):
        """The method plots the calculated loss values for training and validation
        """
        plt.plot(self.train_losses, label="Training loss")
        plt.plot(self.val_losses, label="Validation loss")
        plt.legend()
        plt.title("Losses")
        plt.show()
        plt.close()
```

<br>

### 11-1 Training the model


```python
import torch.optim as optim

input_dim = len(X_train.columns)
output_dim = 1
hidden_dim = 64
layer_dim = 3
batch_size = 64
dropout = 0.2
n_epochs = 20
learning_rate = 1e-3
weight_decay = 1e-6

model_params = {'input_dim': input_dim,
                'hidden_dim' : hidden_dim,
                'layer_dim' : layer_dim,
                'output_dim' : output_dim,
                'dropout_prob' : dropout}

model = get_model('lstm', model_params)

loss_fn = nn.MSELoss(reduction="mean")
optimizer = optim.Adam(model.parameters(), lr=learning_rate, weight_decay=weight_decay)


opt = Optimization(model=model, loss_fn=loss_fn, optimizer=optimizer)
opt.train(train_loader, val_loader, batch_size=batch_size, n_epochs=n_epochs, n_features=input_dim)
opt.plot_losses()

predictions, values = opt.evaluate(
    test_loader_one,
    batch_size=1,
    n_features=input_dim
)
```

    [1/20] Training loss: 0.0141	 Validation loss: 0.0132
    [2/20] Training loss: 0.0096	 Validation loss: 0.0109
    [3/20] Training loss: 0.0088	 Validation loss: 0.0100
    [4/20] Training loss: 0.0087	 Validation loss: 0.0109
    [5/20] Training loss: 0.0083	 Validation loss: 0.0096
    [6/20] Training loss: 0.0076	 Validation loss: 0.0099
    [7/20] Training loss: 0.0074	 Validation loss: 0.0098
    [8/20] Training loss: 0.0070	 Validation loss: 0.0099
    [9/20] Training loss: 0.0067	 Validation loss: 0.0109
    [10/20] Training loss: 0.0066	 Validation loss: 0.0101
    


    
<img src="https://user-images.githubusercontent.com/115082062/229083763-45edc8bc-6d82-4ffa-b9ad-672ebdfc1380.png">

    

<br>

### 11-2. Formating the predictions
이제 예측한 결과값을 스케일링 하기 전으로 되돌려야 한다. 이를 위해 `inverse_transform`을 사용한다.


```python
def inverse_transform(scaler, df, columns):
    for col in columns:
        df[col] = scaler.inverse_transform(df[col])
    return df


def format_predictions(predictions, values, df_test, scaler):
    vals = np.concatenate(values, axis=0).ravel()
    preds = np.concatenate(predictions, axis=0).ravel()
    df_result = pd.DataFrame(data={"value": vals, "prediction": preds}, index=df_test.head(len(vals)).index)
    df_result = df_result.sort_index()
    df_result = inverse_transform(scaler, df_result, [["value", "prediction"]])
    return df_result


df_result = format_predictions(predictions, values, X_test, scaler)
df_result
```





  <div id="df-83dc086f-7475-45ec-a002-91466749227b">
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
      <th>value</th>
      <th>prediction</th>
    </tr>
    <tr>
      <th>Datetime</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-11-14 18:00:00</th>
      <td>1493.000000</td>
      <td>1586.771240</td>
    </tr>
    <tr>
      <th>2015-11-14 19:00:00</th>
      <td>1521.000000</td>
      <td>1594.307617</td>
    </tr>
    <tr>
      <th>2015-11-14 20:00:00</th>
      <td>1505.000000</td>
      <td>1593.699585</td>
    </tr>
    <tr>
      <th>2015-11-14 21:00:00</th>
      <td>1488.000000</td>
      <td>1578.892334</td>
    </tr>
    <tr>
      <th>2015-11-14 22:00:00</th>
      <td>1458.000000</td>
      <td>1546.280518</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2018-08-02 20:00:00</th>
      <td>1966.000000</td>
      <td>2471.009766</td>
    </tr>
    <tr>
      <th>2018-08-02 21:00:00</th>
      <td>1944.000000</td>
      <td>2393.057129</td>
    </tr>
    <tr>
      <th>2018-08-02 22:00:00</th>
      <td>1901.000122</td>
      <td>2286.193604</td>
    </tr>
    <tr>
      <th>2018-08-02 23:00:00</th>
      <td>1789.000000</td>
      <td>2138.349121</td>
    </tr>
    <tr>
      <th>2018-08-03 00:00:00</th>
      <td>1656.000000</td>
      <td>1878.640991</td>
    </tr>
  </tbody>
</table>
<p>23814 rows × 2 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-83dc086f-7475-45ec-a002-91466749227b')"
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
          document.querySelector('#df-83dc086f-7475-45ec-a002-91466749227b button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-83dc086f-7475-45ec-a002-91466749227b');
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

<br>


### 11-3. calculating error metrics

스케일링한 것을 원래 형태로 바꾸어준 후 MAE, RMSE, R^2 메트릭으로 오차를 계산해줄 것이다.


```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

def calculate_metrics(df):
    result_metrics = {'mae' : mean_absolute_error(df.value, df.prediction),
                      'rmse' : mean_squared_error(df.value, df.prediction) ** 0.5,
                      'r2' : r2_score(df.value, df.prediction)}
    
    print("Mean Absolute Error:       ", result_metrics["mae"])
    print("Root Mean Squared Error:   ", result_metrics["rmse"])
    print("R^2 Score:                 ", result_metrics["r2"])
    return result_metrics

result_metrics = calculate_metrics(df_result)
```

    Mean Absolute Error:        167.16405
    Root Mean Squared Error:    204.08217745861592
    R^2 Score:                  0.5153430919596642
    

<br>

### 11-4. Generating Baseline model
일반 선형 회귀 모델을 만들어 점수를 계산하고, 이를 비교해보는 것도 좋은 방법이다.


```python
from sklearn.linear_model import LinearRegression

def build_baseline_model(df, test_ratio, target_col):
    X, y = feature_label_split(df, target_col)
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=test_ratio, shuffle=False
    )
    model = LinearRegression()
    model.fit(X_train, y_train)
    prediction = model.predict(X_test)

    result = pd.DataFrame(y_test)
    result["prediction"] = prediction
    result = result.sort_index()

    return result

df_baseline = build_baseline_model(df_features, 0.2, 'value')
baseline_metrics = calculate_metrics(df_baseline)
```

    Mean Absolute Error:        181.14435507999497
    Root Mean Squared Error:    220.20121350349825
    R^2 Score:                  0.435760267567847
    

<br>

### 11.5. Visualizing the predictions


```python
import plotly.offline as pyo
import plotly.graph_objs as go
from plotly.offline import iplot


def plot_predictions(df_result, df_baseline):
    data = []
    
    value = go.Scatter(
        x=df_result.index,
        y=df_result.value,
        mode="lines",
        name="values",
        marker=dict(),
        text=df_result.index,
        line=dict(color="rgba(0,0,0, 0.3)"),
    )
    data.append(value)

    baseline = go.Scatter(
        x=df_baseline.index,
        y=df_baseline.prediction,
        mode="lines",
        line={"dash": "dot"},
        name='linear regression',
        marker=dict(),
        text=df_baseline.index,
        opacity=0.8,
    )
    data.append(baseline)
    
    prediction = go.Scatter(
        x=df_result.index,
        y=df_result.prediction,
        mode="lines",
        line={"dash": "dot"},
        name='predictions',
        marker=dict(),
        text=df_result.index,
        opacity=0.8,
    )
    data.append(prediction)
    
    layout = dict(
        title="Predictions vs Actual Values for the dataset",
        xaxis=dict(title="Time", ticklen=5, zeroline=False),
        yaxis=dict(title="Value", ticklen=5, zeroline=False),
    )

    fig = dict(data=data, layout=layout)
    iplot(fig)
    
    
# Set notebook mode to work in offline
pyo.init_notebook_mode()

plot_predictions(df_result, df_baseline)
```


