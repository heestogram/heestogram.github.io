---
title: "점프 투 파이썬 공부기록"
excerpt: "점프 투 파이썬 책을 읽고 공부한 기록입니다."

toc: true
toc_label: "목차"
toc_sticky: true

published: true

tags: [python]


categories:
  - Python

date: 2022-11-15 23:50:00 +09:00
last_modified_at: 2022-11-15 23:50:00 +09:00
---
<br>

<div class="notice--primary" markdown="1">
💡 데이터 분석의 툴로서 파이썬을 익히는 것은 기초이자 필수입니다.<br>
군대 사지방에서 틈틈이 ‘점프 투 파이썬’ 책으로 공부했고, 아래는 공부하며 남긴 코드 기록들입니다.<br>
책에 나와있는 예시 코드를 군대나 야구, 축구에 관련된 것들로 변형하여 공부했습니다.<br>
또한 각 챕터 마지막에 수록된 연습문제를 자체적으로 변형하여 만들어보고, 그에 맞는 답을 코딩했습니다.
</div>


<br>

## 02장 파이썬 프로그래밍의 기초

### 02-2 문자열 자료형

```python
#줄바꿈
multiline= """heeh 
hehe
"""
mtl="hehe\nhe"
#print(mtl)

#문자열 합치기
print(multiline+mtl)
#문자열 곱하기
print(mtl*2)

#인덱싱
hj="heejun"
print(hj[3]) #=> j(4번째)
print(hj[-1]) #=> n(뒤에서 1번째)

#슬라이싱
print(hj[1:3]) #=> ee(2번째부터 3번째)
군번="21-70003973"
year=군번[0:2]
nno=군번[-4:]
print(year) #=> 21
print(nno) #=> 3973

#정렬과 공백
txt="%10s heejun" %"hi"
txt="%-10shi" %"hj"
#print(txt)
#소수점 표현
pr="%0.5f" %3.29487572
print(pr)
```

```python

#문자열 포매팅
txt="He have made %d hits" %5
txt="He have made %s" %"doubel-play into the ground"
number=10
name="jm"
txt="i bought %d things for %s" %(number,name)
print(txt)
# %s(문자열) %d(정수) %c(문자1개) %f(부동소수) 

#format함수
stat="Homerun"
number=3
txt="He is leading {0} for {1} days".format(stat,number)
print(txt)
txt="He is leading {stat} for {number} days".format(stat="Strike-out",number=4)
print(txt)
txt="why did you do{0:>10}".format("?") #치환되는 문자열 오른쪽 정렬하고 총자릿수 10으로
txt="why did you do{0:^10}!".format("?") #치환되는 문자열 가운데 정렬하고 총자릿수 10으로
txt="why did you do{0:=^10}".format("?") #치환되는 문자열 가운데 정렬하고 공백은 =, 총자릿수 10으로
txt="{0:0.4f}".format(3.456789) #치환되는 수를 소수점 4자리까지
print(txt)
```

```python

#문자열 내장함수
name=" Wayne Rooney "
a=name.count("e") #문자 개수
a=name.find("e") #문자 위치(존재하지 않는 문자면 -1)
a=name.index("e") #문자 위치(존재하지 않는 문자면 오류)
a="-".join(name) #문자열 삽입
a=name.upper() #소문자를 대문자로
a=name.lower() #대문자를 소문자로
a=name.capitalize() #첫문자를 대문자로
a=name.rstrip() #오른쪽 공백 지우기
a=name.lstrip() #왼쪽 공백 지우기
a=name.strip() #양쪽 공백 지우기
a=name.replace("Wayne","kai") #문자열 대체
a=name.split('e') #입력문자를 기준으로 문자열 나누기
a=name.endswith('y') #입력문자로 문자열이 끝나는지 T/F판별
a=name.startswith('w') #입력문자로 문자열이 끝나는지 T/F판별
print(a)
```

### 02-3 리스트 자료형

```python
#리스트 인덱싱
list=[1,2,"he",[3,4,5,6]]
print(list[3][1]+list[0])
#리스트 슬라이싱
print(list[3][:2])

#리스트 연산
lst=['a','b','c']
print(list+lst)
print(lst*3)
print(len(list))
print(str(list[3][3])+lst[1]) #str:정수나 실수를 문자열로
lst[0]='f' #리스트 수정
del lst[2] #리스트 요소 삭제
```

```python

#리스트 관련 함수들
list.append([7,'a']) #맨 마지막에 입력어 추가
lst.sort() #리스트 정렬
list.reverse() #리스트 역순으로 뒤집기
a=list.index('he') #요소 위치
list.insert(2,'a') #2자리에 'a' 삽입
list.remove('a') #처음으로 나오는 a 제거
list.pop() #맨 마지막 요소 돌려주고 제거
b=list.pop(0) #0자리 요소 돌려주고 제거
print(lst)
print(list)
print(a)
lst=[1,2,3]
print(lst.extend([7,9]))
print(b)
max(lst) #최댓값
min(lst) #최솟값
sum(lst) #리스트 요소 총합
len(lst) #리스트 요소 개수
```

### 02-4 튜플 자료형

```python
#튜플. 튜플은 몇 가지 점을 제외하곤 리스트와 거의 동일.
# 리스트는 [], 튜플은 ()
# 리스트는 그 값의 생성, 삭제, 수정이 가능하지만 튜플은 그 값을 바꿀 수 없다.
# 하지만 튜플도 슬라이싱, 인덱싱, 더하기, 곱하기, len은 가능하다.
# 하나의 데이터를 저장하고 싶으면 쉼표를 입력해야 함. my=(1,)
# 원칙적으로 튜플은 괄호와 함께 데이터를 정의해야 하지만, 사용자 편의를 위해 괄호 없이도 동작함.
```

### 02-5 딕셔너리 자료형

```python
# 딕셔너리의 key는 변하지 않는(immutable) 값, 즉 문자열, 숫자, 튜플만 가능하다. 즉, 리스트는 불가능!
a={'position':"Catcher",'avg':0.301,'ops':0.811}
a['Homerun']=21 #추가
a['team']='kia-tigers'
del a['Homerun']
del a['position']
print(a)

print(a.keys()) #key만 모아서 출력
print(a.values()) #value만 모아서 출력
print(a.items()) #key, value를 튜플로 모아서 출력
print(a.clear()) #딕셔너리 안의 모든 요소 삭제
print(a.get('position')) #who의 value 출력. 없는 key라면 none표시.
print(a['position']) #who의 value 출력. 없는 key라면 오류.
print('position' in a) #position이라는 key가 딕셔너리에 있는지 조사 참이면 true.

batter={'강백호':[0.341,18],'김선빈':[0.307,4],'박병호':[0.256,20]} #딕셔너리의 value는 리스트로 여러개를 묶을 수 있음

key=('강백호','양의지','이정후')
val=(0.395,0.348,0.345)
dict(zip(key,val))   #zip함수를 이용하여 key와 value를 엮어 딕셔너리 만들 수 있음
```

### 02-6 집합 자료형

```python
#  집합(set) 자료형.
# 중복을 허용하지 않는다.
# 순서가 없다(unodered)
 # 중복을 허용하지 않는 set의 특징은 자료형의 중복을 제거하기 위한 필터 역할로 종종 사용
 # 교집합, 합집합, 차집합을 구할 때 유용
s=set("hello")
print(s)
s1=set([1,2,3,4,5,])
s2=set([4,5,6,7])
print(s1&s2) #교집합 = s1.intersection(s2)
print(s1|s2) #합집합 = s1.union(s2)
print(s1-s2) #차집합 = s1.difference(s2)
s1.add(10) #값 1개 추가하기
s1.update([10,11,12]) #값 여러 개 추가하기
s1.remove(1) #특정 값 제거하기
```

### 02-7 불 자료형

```python
#  불 자료형: True와 False를 나타내는 자료형.
print(3>1)
# 문자열, 리스트, 튜플, 딕셔너리 등의 값이 비어있으면("",[],(),{}) 거짓. 숫자는 그 값이 0일때 거짓.
bool("p") # 문자열이 차있으므로 True
bool("") # 문자열이 비었으므로 False
```

### 02장 연습문제

**Q1**

이정후의 5시즌간 안타는 다음과 같다. 이정후의 통산 평균 안타 수를 구하라.

| 시즌 | 안타 수 |
| --- | --- |
| 2017 | 179 |
| 2018 | 163 |
| 2019 | 193 |
| 2020 | 181 |
| 2021 | 167 |

```python
print((179+163+193+181+167)/5)
```

**Q2**

자연수 13이 홀수인지 짝수인지 판별할 수 있는 방법에 대해 말해 보자.

```python
print(13%2==1)
```

**Q3**

군번에서 하이푼 뒤 첫 네 자리는 기수를 의미한다. 군번에서 기수를 나타내는 숫자만 출력해보자.

```python
num="21-70003973"
print(num[3:7])
```

**Q4**

[1, 3, 5, 4, 2] 리스트를 [5, 4, 3, 2, 1]로 만들어 보자.

```python
lst=[1,3,5,4,2]
lst.sort()
```

**Q5**

딕셔너리 pitcher에서 ‘ERA’에 해당되는 value를 추출해보자.

```python
pitcher = {'win':15, 'ERA': 3.24, 'K':182, 'team':'kia'}
print(pitcher.pop('ERA'))
```

**Q6**

inv 딕셔너리에는 key에 선수 이름, value에 평균자책점과 승 기록이 저장되어있다. 아래 물음에 답해보자.

```python
inv={'로켓':[2.38,7,4],'백정현':[2.48,8],'브룩스':[3.35,3]}

# "(선수이름)의 평균자책점은 ~입니다"라는 출력문이 나오도록 해보자
a='브룩스'
print(a+'의 평균자책점은 ',inv[a][0],'입니다.')

#평균자책점 3.87, 4승을 기록한 이의리를 inv에 추가해보자.
inv['이의리']=[3.87,4]
print(inv)

#평균자책점 2.56, 7승을 기록한 수아레즈와 평균자책점 2.80, 7승을 기록한 최원준을 inv에 한꺼번에 추가해보자
sp={'수아레즈':[2.56,7],'최원준':[2.80,7]}
inv.update(sp)

#다음 key와 value를 딕셔너리로 만들어보자.
key=('강백호','양의지','이정후')
val=(0.395,0.348,0.345)
print(dict(zip(key,val)))
```

---

## 03장 프로그램의 구조를 쌓는다! 제어문

### 03-1 if문

```python
# x or y: x와 y 중 하나만 참이어도 참
# x and y: x와 y 모두 참이어야 참
# not x: x가 거짓이면 참
# x in 리스트, 튜플, 문자열
# pass: 아무런 값도 출력하지 않음

player=['박지성','루니','테베즈']

    
if '호날두' in player:
    print('개시')
elif '치차리토' in player:
    print('개시')
else:
    print('불발')

#조건부 표현식: 참일 경우 출력값 if 조건문 else
```

### 03-2 while문

```python

seed=10000

while seed>0:
    expense= int(input("지출액은?: "))
    seed=seed-expense
    print("잔액은 %d원입니다." %seed)
    if seed<expense:
        print("잔액이 없습니다!")
        break #강제종료

a=0
while a<10:
    a=a+1
    if a%2==0:
        continue
    print(a)
```

### 03-3 for문

```python
fs=['기상대','항작전본','운항관제대','255대대','256대대']
for a in fs:
    print("항작전대 %s" %a)

a=[(1,3),(3,5),(5,7)]
for (f,l) in a:
    print(f*l)

stat=[40,50,70,90]
a=[1,2,3,4]
result=[]
for i in a:
    j=i*3
    result.append(j)
print(result)
number=0

for i in stat:
    number=number+1
    a="{0}번학생은 {1}점으로 {2}입니다."
    if i<60:
        continue        
    print(a.format(number,i,'합격'))
```

```python
a=0
for i in range(11):
    a=a+i
print(a)

for i in range(2,11):
    for j in range(2,11):
        print(i*j, end=' ') #매개변수 end를 넣으면 같은 줄에 계속
    print('')

a=[1,2,3,4]
result=[]
for i in a:
    j=i*3
    result.append(j)
print(result)
```

```python
# 리스트내포
a=[1,2,3,4]
result=[i*3 for i in a]
print(result)

#짝수만 담고 싶다면?
a=[1,2,3,4]
result=[i*3 for i in a if i%2==0]
print(result)

```

### 03장 연습문제

**Q1**

while문을 사용해 1부터 1000까지의 자연수 중 3의 배수의 합을 구해 보자.

```python
n=0
a=0
while n<1001:
    n=n+1
    if n%3==0:
        a=a+n
print(a)
```

**Q2**

while문을 사용하여 다음과 같이 별(``)을 표시하는 프로그램을 작성해 보자.

```python
a=''
while len(a)<5:
a=a+'*'
print(a)
```

**Q3**

한 야구팀에 총 10명의 타자가 있다. 이 타자들의 시즌 안타 수는 다음과 같다. for문을 사용하여 이 팀 타자들의 평균 안타 수를 구해 보자.

```python
b=0
a=[73,89,55,76,141,168,102,80,89,187]
for i in a:
    b=i+b
print(b/len(a))
```

**Q4**

리스트 중에서 홀수에만 2를 곱하여 저장하는 다음 코드가 있다. 이 코드를 리스트 내포(list comprehension)를 사용하여 표현해 보자.

```python
number=[1,2,3,4,5]
result=[i*2 for i in number if i%2!=0]
print(result)
```

**Q5**

pl리스트에는 한 투수의 ERA(평균자책점), WIN(승), LOSE(패), WHIP(이닝당 출루 허용률) 기록이 저장되어있다. 아래 물음에 답해보자.

```python
pl=[['ERA','WIN','LOSE','WHIP'],[2.34,13,5,1.08],[3.46,11,8,1.21],[4.01,7,8,1.30]]

#whip만 출력하기
pl=[['ERA','WIN','LOSE','WHIP'],[2.34,13,5,1.08],[3.46,11,8,1.21],[4.01,7,8,1.30]]
for a in pl[1:]:
    print(a[3])

#ERA 3.0이상만 출력
for a in pl[1:]:
    if a[0]>3:
        print(a[0])

#승이 패보다 적은 경우 출력
for a in pl[1:]:
    if a[1]>a[2]:
        print(a[1])

#승률을 Pro리스트에 저장
pro=[]
for a in pl[1:]:
    pro.append(round(a[1]/(a[1]+a[2]),2))
print(pro)
```

---

## 04장 프로그램의 입력과 출력은 어떻게 해야 할까?

### 04-1 함수

```python
#일반적인 함수
def mp(a,b):
    return a*b
print(mp(3,4))

#입력값이 없는 함수
def say():
    return 'hello'
print(say())

#결과값이 없는 함수
def add(a,b):
    print("%d, %d의 합은 %d입니다"%(a,b,a+b))
add(3,4)
#출력문장은 있지만 return값이 없으니 결과값이 없다는 것임. 애초에 함수 자체에 print기능까지 넣었으니 값을 출력하고 싶을 때 print(add(3,4))할 필요 없이 add(3,4)만 하면 됨.

#입력값, 결과값이 모두 없는 함수
def say2():
    print('hi')
say2()
```

```python
#입력값을 여러개 만들고 싶은 경우
#매개변수 앞에 *을 넣어주면 됨!
#아래는 여러 매개변수를 모두 합하는 함수.
def addm(*a):
    result=0
    for i in a:
        result+=i
    print(result)
addm(1,2,3)

def addmul(choice,*a):
    if choice=='add':
        result=0
        for i in a:
            result+=i
    elif choice=='mul':
        result=1
        for i in a:
            result*=i
    print(result)

addmul('mul',3,4,5)
```

```python
#키워드 파라미터(딕셔너리 만들기)
def kwargs(**k):
    print(k)
kwargs(양현종='kia',오재일='삼성',문동주='한화')

#함수의 결과값은 언제나 하나이다
def addmul(a,b):
    return a+b, a*b
result= addmul(3,4)
print(result) #=> 두 개의 값이 하나의 튜플로 나옴.(7,12)

result1, result2=addmul(3,4)
print(result1) #=> 7
print(result2) #=>12
```

```python
#return의 또다른 쓰임새. 특정 조건에서 함수를 빠져나가고 싶으면 return을 단독으로 사용하면 됨
def spnotice(sp):
    if sp=='브룩스':
        return
    print('기아의 선발투수는 %s입니다'%sp)
spnotice('브룩스') #아무 값도 안 나옴
spnotice('멩덴') #기아의 선발투수는 멩덴입니다.

#매개변수에 초깃값 미리 설정하기
def saymy(name,old,man=True):
    print('나의 보직은 %s입니다.'%name)
    print('기수는 %d기입니다.'%old)
    if man:
        print('남자입니다')
    else:
        print('여자입니다')
saymy('행정병',22)
```

```python
#함수 안에서 선언한 매개변수는 함수 밖의 변수로 쓰이지 않는다
a=1
def test(a):
    a=a+1
print(a) #결과값으로 2가 아닌 1이 나옴.

#함수 안에서 함수 밖의 변수를 변경하는 방법
a=1
def test(a):
    a+=1
    return a
a=test(a)
print(a)

a=1
def test():
    global a #global은 함수 밖의 변수를 함수 안에서 사용하겠다는 명령어.
    a+=1
test()
print(a)

#lambda: 함수를 생성하는 예약어로 def와 동일한 역할을 한다. 보통 함수를 한줄로 간결하게 만들 때 사용한다. 
add= lambda a,b:a+b
result=add(3,4)
print(result)
```

### 04-2 사용자 입력과 출력

```python
sp=input('오늘의 선발투수는: ')
print('오늘의 선발투수는 %s입니다'%sp)
#input은 입력되는 모든 것을 문자열로 취급한다.

#print문에서 큰따옴표로 둘러싸인 문자열은 +연산과 동일하다
print("life""is""too short")
#콤마는 문자열 띄어쓰기다
print('life','is','too short')
```

### 04-3 파일 읽고 쓰기

```python
f=open('새파일.txt','w') #r(읽기모드) w(쓰기모드), a(추가모드: 파일 마지막에 새 내용을 추가)
f.close() #열려 있는 파일 객체를 닫아주는 역할

#프로그램의 외부에 저장된 파일을 읽는 여러 방법
#readline 함수
f = open("C:/doit/새파일.txt", 'r')
line = f.readline()
print(line)
f.close()

#readlines 함수
f = open("C:/doit/새파일.txt", 'r')
lines = f.readlines()
for line in lines:
    print(line)
f.close()

#read 함수
f = open("C:/doit/새파일.txt", 'r')
data = f.read()
print(data)
f.close()

#with문은 파일을 열고 닫는 것을 자동으로 처리해준다
with open("foo.txt", "w") as f:
    f.write("Life is too short, you need python")
```

### **04장 연습문제**

**Q1**

주어진 자연수가 홀수인지 짝수인지 판별해주는 함수(is_odd)를 작성해보자.

```python
def is_odd(a):
    if a%2==0:
        print('짝수')
    else:
        print('홀수')
is_odd(2)
```

**Q2**

문자열과 한줄에 출력될 글자 수를 입력을 받아 한 줄에 입력된 글자 수만큼 출력하는 printn 함수를 작성하라.

```python
def printn(a,b):
    num=len(a)//b
    for i in range(num+1):
        print(a[i*b:i*b+b])
printn('전역이답이다휴가나가고싶어',3)
```

**Q3**

숫자로 구성된 하나의 리스트를 입력받아, 짝수들을 추출하여 리스트로 반환하는 picke 함수를 구현하라.

```python
def picke(a):
    a=[]
    for i in a:
        if i%2==0:
            a.append(i)
    print(a)
picke([1,2,3,4,5,6])
```

---

## 05장 파이썬 날개달기

### 05-1 클래스

```python
# 과자틀=클래스(class)  과자=객체(objet)
# 동일한 클래스로 만든 객체들은 서로 전혀 영향을 주지 않는다.

#사칙연산 클래스 만들기
#클래스 내부의 함수를 메서드라 함.
class fourcal:
    def setdata(self, first, second):
        self.first=first
        self.second=second
    def add(self):
        result=self.first+self.second
        return result
    def mul(self):
        result=self.first*self.second
        return result
    def sub(self):
        result=self.first-self.second
        return result
    def div(self):
        result=self.first/self.second
        return result
    
a=fourcal()
b=fourcal()
a.setdata(4,2)
b.setdata(6,3)
print(a.add()) #6
print(a.mul()) #8
print(a.sub()) #2
print(a.div()) #2

print(b.add()) #9
print(b.mul()) #18
print(b.sub()) #3
print(b.div()) #2
```

```python
#생성자(constructor)
#setdata 없이 객체에 초기값을 설정하는 것.
class fourcal:
    def __init__(self, first, second):
        self.first=first
        self.second=second
    def add(self):
        result=self.first+self.second
        return result
    def mul(self):
        result=self.first*self.second
        return result
    def sub(self):
        result=self.first-self.second
        return result
    def div(self):
        result=self.first/self.second
        return result

a=fourcal(4,2)
print(a.add())
```

```python
#클래스 상속
#클래스를 상속하기 위해서는 클래스 이름 뒤 괄호 안에 상속할 클래스 이름을 넣으면 됨
#상속은 기존 클래스를 변경하지 않고 기능을 추가하거나 수정하려고 할 때 사용함.
class morefourcal(fourcal):
    def pow(self):
        result=self.first**self.second
        return result

a=morefourcal(3,4)
print(a.pow())
```

```python
#메서드 오버라이딩(덮어쓰기)
class safefourcal(fourcal):
    def div(self):
        if self.second==0:
            return 0
        else:
            return self.first/self.second
        
a=safefourcal(3,0)
print(a.div())
```

### 05-2 모듈

### 05-3 패키지

### 05-4 예외처리

```python
try:
    4/0
    a=[1,2,3]
    print(a[3])
except ZeroDivisionError as e:
    print(e)
except IndexError as e:
    print(e)
#0나누기 오류가 먼저 발생했으므로 인덱스 오류는 출력되지 않음.

#try문에 else절 사용하기
try:
    age=int(input('기수를 입력하세요: '))
except:
    print('부정확한 입력입니다')
else:
    if age<=815:
        print('병장입니다')
    elif age<=822:
        print('상병입니다')
    elif age<=828:
        print('일병입니다')
    else:
        print('이병입니다')
```

### 05장 연습문제

**Q1**

투수(pitcher) 클래스에 (팀, 삼진, 던지는 팔)을 받는 생성자를 추가해보자.

```python
class pitcher:
    def __init__(self,team,k,arm):
        self.team=team
        self.k=k
        self.arm=arm

이의리=pitcher("기아타이거즈",97,'좌완')
print(이의리.name)
```

**Q2**

투수(pitcher) 클래스에 (팀, 삼진, 던지는 팔)을 받는 setinfo 메소드를 추가해보자.

```python
class pitcher:
    def __init__(self,team,k,arm):
        self.team=team
        self.k=k
        self.arm=arm
    def who(self):
        print("팀:{0} 삼진:{1} 던지는 팔:{2}".format(self.team,self.k,self.arm))
    def setinfo(self,team,k,arm):
        self.team=team
        self.k=k
        self.arm=arm

이의리 = pitcher("무소속", 0, "모름")
이의리.who()     

이의리.setInfo("nc다이노스", 155, "좌완")
이의리.who()      # Human.who(areum)
```

