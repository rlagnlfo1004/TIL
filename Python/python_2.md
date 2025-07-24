
## 10. 함수 (Function)

### 디폴트(기본) 값 매개변수

함수 정의 시 매개변수에 기본값을 지정할 수 있습니다. 기본값을 갖는 매개변수는 반드시 기본값이 없는 매개변수 뒤에 와야 합니다.

```python
def have_default_value(n1, n2, n3 = "df1", n4 = "df2"):
    print(n1, n2, n3, n4)

have_default_value(1, 2, 3, 4) # 결과: 1 2 3 4
have_default_value(1, 2)       # 결과: 1 2 df1 df2
```
<br><br>

### 함수의 매개변수 참조 관계

파이썬은 매개변수를 위해 별도의 메모리 공간을 할당하지 않고, 전달된 객체에 새로운 이름(매개변수명)을 붙이는 방식으로 처리합니다. 따라서 리스트와 같은 변경 가능한(mutable) 객체가 함수에 전달되면, 함수 내에서 해당 객체를 변경하면 원본 객체도 변경됩니다.

즉, `1, 2, 3 - 변수st` 와 같은 메모리 공간에 `s` 라는 이름 하나 더 주는 것이다.
<br><br>

```python
def func(s):
    s[0] = 0
    s[-1] = 0

st = [1, 2, 3]
func(st)
print(st) # 결과: [0, 2, 0]
```
<br><br>

## 11. 모듈 (Module)

모듈은 함수, 변수, 클래스 등을 담고 있는 파이썬 파일입니다. 재사용 가능한 코드 단위를 제공하여 코드의 구성과 관리를 용이하게 합니다.
<br><br>

### 모듈 만들어 보기

```python
#circle.py
PI = 3.14

def ar_circle(rad):
		return rad * rad * PI

def ci_circle(rad):
		return rad * 2 * PI
```

이렇게 해서 만들어진 하나의 소스파일을 가리켜 ‘모듈(module)’이라 한다. 참고로 ‘모듈’이라 하면, 이렇듯 필요할 때 가져다 쓸 수 있는, 또는 다른 프로그램의 일부가 될 수 있는 내용을 담고 있는 파일을 의미한다. 보편적으로 파이썬의 모든 소스파일을 그냥 ‘모듈’이라 부르는 경우도 많다.
<br><br>

### 모듈을 가져다 쓰는 방법 - #1  `import`

```python
# main.py
import circle # circle.py 모듈을 가져옴

def main():
    r = float(input("반지름 입력: "))
    ar = circle.ar_circle(r)
    print("넓이:", ar)
    ci = circle.ci_circle(r)
    print("둘레:", ci)

main()
```
<br><br>

특정 함수만 가져와서 모듈명 없이 바로 사용합니다.

```python
# main.py
from circle import ar_circle    # circle.py에서 ar_circle 함수를 가져옴
from circle import ci_circle    # circle.py에서 ci_circle 함수를 가져옴

def main():
    r = float(input("반지름 입력: "))
    ar = ar_circle(r) # 모듈명 없이 바로 사용
    print("넓이:", ar)
    ci = ci_circle(r) # 모듈명 없이 바로 사용

main()
```
<br><br>

### 모듈을 가져다 쓰는 방법 - #2 `as` (별칭 지정)

```python
# main.py
import circle as cc # circle 모듈을 cc라는 별칭으로 가져옴

def main():
    r = float(input("반지름 입력: "))
    ar = cc.ar_circle(r) # 별칭을 사용하여 접근
    print("넓이:", ar)
    ci = cc.ci_circle(r) # 별칭을 사용하여 접근

main()
```
<br><br>

```python
# main.py
from circle import ar_circle as ac # ar_circle 함수를 ac라는 별칭으로 가져옴
from circle import ci_circle as cc # ci_circle 함수를 cc라는 별칭으로 가져옴

def main():
    r = float(input("반지름 입력: "))
    ar = ac(r) # 별칭을 사용하여 바로 사용
    print("넓이:", ar)
    ci = cc(r) # 별칭을 사용하여 바로 사용

main()
```
<br><br>

## 12. 딕셔너리 (Dictionary)

딕셔너리는 `키(key): 값(value)` 쌍으로 이루어진 데이터를 저장하는 자료형입니다. 키는 중복될 수 없지만, 값은 중복될 수 있습니다. 리스트는 키로 사용할 수 없습니다.

```python
dc = {'정은호': '010-3333-56xx', '구아나': '010-2222-65xx'}

dc_items = {
    '코카콜라': (900, '탄산음료'),
    '바나나우유': (750, '유제품'),
    '비타500': (600, '비타민음료'),
    '삼다수': (450, '생수')
}
```
<br><br>

### 딕셔너리의 데이터 참조, 수정, 추가, 삭제

```python
dc = {
    '코카콜라': 900,
    '바나나우유': 750,
    '비타500': 600,
    '삼다수': 450
}

# 데이터 참조
v = dc['삼다수']
print(v) # 결과: 450

# 데이터 수정 (1)
dc['삼다수'] = 550
print(dc) # 결과: {'코카콜라': 900, '바나나우유': 750, '비타500': 600, '삼다수': 550}

# 데이터 수정 (2)
dc['삼다수'] += 200
print(dc) # 결과: {'코카콜라': 900, '바나나우유': 750, '비타500': 600, '삼다수': 750}

# 데이터 추가
dc['카페라떼'] = 1300
print(dc) # 결과: {'코카콜라': 900, '바나나우유': 750, '비타500': 600, '삼다수': 750, '카페라떼': 1300}

# 삭제
del dc['비타500']
print(dc) # 결과: {'코카콜라': 900, '바나나우유': 750, '삼다수': 750, '카페라떼': 1300}
```
<br><br>

### 연산자 == 와 딕셔너리

```python
t1 = [1, 2, 3]
t2 = [1, 2, 3]
t3 = [3, 2, 1]

print(t1 == t2) # 결과: True
print(t1 == t3) # 결과: False (리스트는 순서가 중요)

d1 = {1: 'a', 2: 'b'}
d2 = {1: 'a', 2: 'b'}
d3 = {2: 'b', 1: 'a'} # 키와 값 쌍은 d1, d2와 동일하지만 순서가 다름

print(d1 == d2) # 결과: True
print(d1 == d3) # 결과: True (딕셔너리는 순서가 중요하지 않음)
```
<br><br>

### 딕셔너리의 `for` 루프

딕셔너리를 `for` 루프에 사용하면 기본적으로 키(key)를 반복합니다.

```python
dc ={
    '코카콜라': 900,
    '바나나우유': 750,
    '비타500': 600,
    '삼다수': 450
}

for i in dc:
    print(i, end = ' ') # 결과: 코카콜라 바나나우유 비타500 삼다수

for i in dc:
    dc[i] += 70 # 각 키의 값에 70을 더함

print(dc) # 결과: {'코카콜라': 970, '바나나우유': 820, '비타500': 670, '삼다수': 520}
```
<br><br>

## 13. 클래스와 객체 (Class and Object)

### 지역변수와 전역변수

- **지역변수:** 함수 내에서 선언되며 해당 함수 내에서만 유효합니다.
- **전역변수:** 함수 외부에서 선언되며 프로그램 전체에서 접근 가능합니다.
<br><br>

`global` 키워드를 사용하여 함수 내에서 전역변수를 수정할 수 있습니다. 전역변수에 값을 참조만 할 경우 `global` 선언은 필수가 아니지만, 명시적으로 `global`을 붙이면 해당 변수가 전역변수임을 쉽게 파악할 수 있어 코드를 읽기 편하게 합니다.
<br><br>

```python
cnt = 100 # 전역변수

def func_local():
    cnt = 0 # 지역변수 cnt (전역변수 cnt와 다른 변수)
    print(cnt)

func_local() # 결과: 0
print(cnt)   # 결과: 100 (전역변수 cnt는 변경되지 않음)
```
<br><br>
```python
cnt = 100 # 전역변수

def func_global():
    global cnt # 전역변수 cnt를 사용하겠다고 선언
    cnt = 0    # 전역변수 cnt를 0으로 수정
    print(cnt)

func_global() # 결과: 0
print(cnt)    # 결과: 0 (전역변수 cnt가 변경됨)
```
<br><br>

다시 한번 말하면, 원하는 것이 지역변수의 선언이 아니라, 전역변수에 0을 저장하는 것이라면, global로 시작을 해야한다. **가급적 함수 내에서 전역변수에 접근하는 상황이라면, global 선언을 넣어주자!!** 이 선언이 필요없는 상황(그냥 값을 참조만 하는 상황)이더라도 말이다. 이를 통해 접근하는 변수가 어떤 변수인지 쉽게 파악이 가능하다.
<br><br>

### 객체 지향 프로그래밍 (OOP)

OOP는 데이터를 속성(인스턴스 변수)으로, 데이터를 조작하는 함수를 메서드로 묶어 `클래스`라는 "설계도"를 만들고, 이 설계도를 기반으로 `객체(인스턴스)`를 생성하여 사용하는 프로그래밍 패러다임입니다.
<br><br>

**OOP 이전의 예시 (`family_age.py`)**

```python
# family_age.py
fa_age = 39 # 전역변수

def up_fa_age():
    global fa_age
    fa_age += 1

def get_fa_age():
    return fa_age

ma_age = 35 # 전역변수

def up_ma_age():
    global ma_age
    ma_age += 1

def get_ma_age():
    return ma_age

def main():
    print("2019년 ... ")
    print("아빠:", get_fa_age())
    print("엄마:", get_ma_age())
    print("2020년 ... ")
    up_fa_age()
    up_ma_age()
    print("아빠:", get_fa_age())
    print("엄마:", get_ma_age())

main()
# 결과:
# 2019년 ...
# 아빠: 39
# 엄마: 35
# 2020년 ...
# 아빠: 40
# 엄마: 36
```

이전 방식은 각 사람의 나이를 별도의 전역변수와 함수로 관리해야 하여 데이터와 기능이 분리되어 있었습니다.
<br><br>

### 클래스와 객체 활용 (`class_object.py`)

클래스를 사용하면 데이터(나이)와 그 데이터를 조작하는 함수(나이 증가, 나이 가져오기)를 하나의 단위로 묶을 수 있습니다.

```python
# class_object.py
class AgeInfo:
    def up_age(self):
        self.age += 1 # self.age는 인스턴스 변수

    def get_age(self):
        return self.age # self.age는 인스턴스 변수

def main():
    fa = AgeInfo() # AgeInfo 클래스의 객체(인스턴스) 생성
    fa.age = 39 # 인스턴스 변수 age 초기화

    print("현재 아빠 나이 ...")
    print("아빠:", fa.get_age())

    print("1년 뒤 ...")
    fa.up_age()
    print("아빠:", fa.get_age())

main()
# 결과:
# 현재 아빠 나이 ...
# 아빠: 39
# 1년 뒤 ...
# 아빠: 40
```

`fa = AgeInfo()` 문장이 실행되면, `AgeInfo` 클래스(설계도)를 바탕으로 `fa`라는 객체(인스턴스)가 생성됩니다. 이 객체는 `age`라는 인스턴스 변수와 `up_age()`, `get_age()` 메서드를 가집니다.
<br><br>

### **변수 fa**

위의 코드를 분석하면, **AgeInfo의 인스턴스가 생성되어서 변수 fa에 저장되었다.**

```python
fa = AgeInfo()       # AgeInfo 클래스(설계도)의 객체를 생성하고 이를 변수 fa 에 저장
```

그렇다면, 변수 fa에 저장된 객체의 모습으로는

다음과 같다. ➡️➡️➡️ 

```python
age

def up_age():
		age += 1
		
def get_age():
		return age
```
<br><br>

1. **인스턴스 변수 : 파이썬이 자동으로 변수 age를 넣어준다.**
    
    ```python
    fa.age = 39
    ```
    
    - 실제로는 위의 문장이 실행되면 fa에 저장된 객체의 변수 age가 생성되고, 39가 저장된다.
2. **인스턴스 메서드**
    
    ```python
    fa.up_age()
    fa.get_age()
    ```
    
    - 매개변수 self를 위해 어떤 값도 전달하지 않는다.
    
<br><br>

### `self` 매개변수

메서드 정의 시 첫 번째 매개변수로 `self`를 사용합니다. `self`는 해당 메서드를 호출한 객체 자신을 가리킵니다. 파이썬은 메서드 호출 시 `self`에 해당 객체를 자동으로 전달합니다.



```python
# family_age.py
class AgeInfo:
    def up_age(self): # self는 이 메서드를 호출한 객체 자신
        self.age += 1

    def get_age(self):
        return self.age

def main():
    fa = AgeInfo()
    ma = AgeInfo()
    me = AgeInfo()

    fa.age = 39 # fa 객체의 age 변수
    ma.age = 35 # ma 객체의 age 변수
    me.age = 12 # me 객체의 age 변수

    sum_ages = fa.get_age() + ma.get_age() + me.get_age()
    print("가족 나이 합:", sum_ages)

    fa.up_age() # AgeInfo.up_age(fa) 와 동일
    ma.up_age() # AgeInfo.up_age(ma) 와 동일
    me.up_age() # AgeInfo.up_age(me) 와 동일

    sum_ages = fa.get_age() + ma.get_age() + me.get_age()
    print("1년 후의 가족 나이 합:", sum_ages)

main()
# 결과:
# 가족 나이 합: 86
# 1년 후의 가족 나이 합: 89
```
<br><br>

### **파이썬이 객체를 관리하는 형태**

<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/3e72d15f-9191-48cb-b260-d89e6fb463cb" />

이는 둘 이상의 AgeInfo 객체들이 다음의 두 함수를 공유하는 모습이다.

```python
def up_age(self):
		self.age += 1
		
def get_age(self):
		return self.age
```

```python
fa.up_age()    # 변수 fa에 저장된 객체의 up_age 함수 호출
```
<br><br>
위의 함수를 실행하면, 우리 눈에는 보이지 않지만, 파이썬은 AgeInfo 클래스의 함수가 실제로 저장된 위치로 가서 **up_age() 함수를 호출하되, fa를 인자로 전달**하면서 다음과 같은 형태로 호출한다.

```python
AgeInfo.up_age(fa)     # up_age 함수 호출하며 fa를 인자로 전달
```
<br><br>

추가로 우리가 직접 함수를 호출 할 수도 있다. 즉, 다음 두 메서드는 동일한 기능의 문장이다.

```python
fa.up_age()                 # 권장 O
AgeInfo.up_age(fa)          # 권장 X ( self의 의미 파악 목적 )
```

`fa.up_age()`와 `AgeInfo.up_age(fa)`는 동일한 기능을 합니다. 전자는 "객체 관점"의 호출이고, 후자는 "클래스 관점"의 호출입니다. 일반적으로 객체 관점의 호출이 권장됩니다.
<br><br>

### `self` 이외의 매개변수를 갖는 메서드

`self` 외에도 일반적인 함수처럼 여러 매개변수를 가질 수 있습니다.

```python
# family_age2.py
class AgeInfo:
    def up_age(self):
        self.age += 1

    def get_age(self):
        return self.age

    def set_age(self, new_age): # self 다음의 new_age는 매개변수
        self.age = new_age # self.age는 인스턴스 변수, new_age는 매개변수

def main():
    fa = AgeInfo()
    fa.set_age(39) # self에 fa가 자동으로 전달되고, new_age에 39가 전달됨
    fa.up_age()
    print("1년 후의 아빠:", fa.get_age())

main()
# 결과:
# 1년 후의 아빠: 40
```

self에는 파이썬이 알아서 전달하니, 두 번째 이후의 매개변수에만 값을 전달하면 된다.

주의 할 점은 ‘매개변수’와 ‘인스턴스 변수’의 이름이 같을 때 이를 구분하는 것이다.
<br><br>

```python
def set_age(self, age):
				self.age = age
```

- age는 매개변수
- self.age는 인스턴스 변수
<br><br>

## 14. 생성자 (`__init__`)

객체의 생성 후에 반드시 해줘야 하는 일이 하나 있다.

**“인스턴스 변수의 초기화”**

파이썬은 객체의 생성과 변수의 초기화를 동시에 진행할 수 있도록 ‘생성자(constructor)’라는 것을 제공한다. 생성자는 객체가 생성될 때 자동으로 호출되는 특별한 메서드입니다. 주로 인스턴스 변수를 초기화하는 데 사용됩니다. `__init__`이라는 이름을 가집니다.
<br><br>

```python
def main():
		fa = AgeInfo()
		fa.age = 20
```

```python
# ctor1.py
class Const:
    def __init__(self): # 생성자, 객체 생성 시 자동 호출
        print("new~")

def main():
    o1 = Const() # 객체 생성 시 __init__ 호출 -> "new~" 출력
    o2 = Const() # 객체 생성 시 __init__ 호출 -> "new~" 출력

main()
# 결과:
# new~
# new~
```
<br><br>

생성자도 다른 메서드와 마찬가지로 `self`를 첫 번째 매개변수로 가집니다. 추가적인 매개변수를 통해 객체 생성 시 값을 전달받아 인스턴스 변수를 초기화할 수 있습니다.

```python
# ctor2.py
class Const:
    def __init__(self, n1, n2): # n1, n2는 매개변수
        self.n1 = n1       # self.n1은 인스턴스 변수
        self.n2 = n2       # self.n2는 인스턴스 변수

    def show_data(self):
        print(self.n1, self.n2)

def main():
    o1 = Const(1, 2) # 1, 2가 __init__의 n1, n2로 전달됨
    o2 = Const(3, 4) # 3, 4가 __init__의 n1, n2로 전달됨
    o1.show_data()   # 결과: 1 2
    o2.show_data()   # 결과: 3 4

main()
```
<br><br>

## 15. 예외처리 (Exception Handling)

예외 처리는 프로그램 실행 중 발생할 수 있는 오류(예외)를 미리 예상하고 적절하게 처리하여 프로그램이 비정상적으로 종료되는 것을 방지하는 기법입니다. `try`, `except`, `finally` 키워드를 사용합니다.
<br><br>

### **`try ~ except`**

```python
try:
    # 예외가 발생할 가능성이 있는 코드
    # (try 영역)
    age = int(input("나이를 입력하세요: "))
    print("입력하신 나이는 다음과 같습니다:", age)
except ValueError:
    # ValueError 예외가 발생했을 때 실행되는 코드
    # (except 영역)
    print("입력이 잘못되었습니다.")

print("만나서 반갑습니다.") # 예외 발생 여부와 상관없이 실행
```
<br><br>

### 보다 적극적인 예외 처리

`while` 루프와 결합하여 유효한 입력이 들어올 때까지 반복적으로 입력을 요청할 수 있습니다. except 영역이 실행되면, while 루프를 반복하여 정상적인 입력이 없으면, 프로그램 사용자에게 입력을 계속 요구합니다.

```python
def main():
    print("안녕하세요.")

    while True: # 유효한 입력이 들어올 때까지 반복
        try:
            age = int(input("나이를 입력하세요: "))
            print("입력하신 나이는 다음과 같습니다:", age)
            break # 정상적으로 입력받으면 루프 종료
        except ValueError:
            print("입력이 잘못되었습니다.")

    print("만나서 반갑습니다.") # 루프 종료 후 실행
main()
```
<br><br>

### 둘 이상의 예외처리

여러 종류의 예외에 대해 각각 다른 처리 로직을 정의할 수 있습니다.

```python
def main():
    bread = 10
    try:
        people = int(input("몇 명? "))
        print("1인당 빵의 수: ", bread / people) # ZeroDivisionError 발생 가능
        print("맛있게 드세요.")
    except ValueError: # 숫자가 아닌 입력 시
        print("입력이 잘못되었습니다.")
    except ZeroDivisionError: # 0으로 나눌 시
        print("0으로는 나눌 수 없습니다.")

main()
```
<br><br>

### 예외 메시지 출력하기

`as` 키워드를 사용하여 발생한 예외 객체를 변수에 담아 예외 메시지를 출력할 수 있습니다. 변수 msg에 오류 메시지가 담깁니다.

```python
def main():
		bread = 10
		try:
				people = int(input("몇 명? "))
				print("1인당 빵의 수: ", bread/people)
				print("맛있게 드세요.")
		except ValueError as msg:             # 변수 msg에 오류 메시지가 담긴다.
				print("입력이 잘못되었습니다.")
				print(msg)
		except ZeroDivisionError as msg:      # 변수 msg에 오류 메시지가 담긴다.
				print("0으로는 나눌 수 없습니다.")
				print(msg)

main()
```
<br><br>

### `finally` 블록

`finally` 블록은 `try` 영역으로 진입하면 예외 발생 유무와 상관없이 **무조건 실행**됩니다. 주로 리소스 해제(파일 닫기 등)와 같은 최종 작업을 수행할 때 유용합니다. `except` 블록 없이 `finally`만 사용할 수도 있습니다.

```python
def main():
    print("안녕하세요.")

    while True:
        try:
            age = int(input("나이를 입력하세요: "))
            print("입력하신 나이는 다음과 같습니다:", age)
            break
        except ValueError:
            print("입력이 잘못되었습니다.")
        finally: # 예외 발생 여부와 상관없이 실행됨
            print("어쨌든 프로그램은 종료합니다.")

main()
```
<br><br>

### 모든 예외 처리

특정 예외 타입을 명시하지 않고 `except`만 사용하면 모든 종류의 예외를 처리할 수 있습니다.

```python
def main():
    bread = 10
    try:
        people = int(input("몇 명? "))
        print("1인당 빵의 수: ", bread / people)
        print("맛있게 드세요.")
    except: # 어떤 종류의 예외든 모두 처리
        print("예외가 발생했습니다.")

main()
```
<br><br>

**주의:** `except: pass`는 모든 예외를 무시하고 아무것도 하지 않습니다. 이는 디버깅을 어렵게 하고 예상치 못한 동작을 유발할 수 있으므로, **정말 필요한 경우가 아니라면 사용하지 않는 것이 좋습니다.**

```python
try:
    # try 영역 ...
    pass
except: # 이렇게 하면 모든 예외가 다 걸려든다.
    pass # pass라고 써 놓으면 아무 일도 하지 않는다.
```