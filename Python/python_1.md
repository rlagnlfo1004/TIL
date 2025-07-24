## 1. eval() 함수

```python
>>> year = eval(input("This year: "))
This year: 2020

>>> year = year + 1
>>> print("Next year: ", year)
Next year: 2021
```
<br><br>

`eval()`은 강력하지만, 위험할 수 있다. eval 은 evaluate의 앞 부분을 딴 것으로, 그 이름처럼 전달된 문자열의 내용을 평가 및 해석해서 무엇을 할지 결정한다. `eval()` 함수는 입력받은 문자열을 파이썬 코드로 평가하고 실행합니다.

```python
>>> result = eval(input("뭐든 넣어요: "))
뭐든 넣어요: 2 - 4 * 5 + 3

>>> print(result)
-15
```
<br><br>

**주의:** `eval()`은 강력하지만, 전달된 문자열이 악의적인 코드일 경우 보안에 매우 취약해질 수 있습니다. 따라서 `eval()` 함수의 사용은 **가급적 제한**해야 합니다.

```python
>>> def ret():
			return 12

>>> result = eval(input("뭐든 넣어요: ")
뭐든 넣어요: ret()

>>> print(result)
12
```
<br><br>

## 2. 반복문(for, while)

### for 반복문

```python
for i in [1, 3, 5]:
    print(i, end = ' ') # 결과: 1 3 5

# range() 함수를 이용한 반복
# range(시작, 끝+1)
for i in range(1, 11): # 1부터 10까지
    print(i, end = ' ') # 결과: 1 2 3 4 5 6 7 8 9 10

# range(끝)은 0부터 끝-1까지
for i in range(3): # 0부터 2까지
    print(i, end = ' ') # 결과: 0 1 2

# 리스트/문자열과 for 문
for i in [1, 2, 3]:
    print(i, end = ' ') # 결과: 1 2 3

for i in 'Happy':
    print(i, end = ' ') # 결과: H a p p y
```
<br><br>

### while 반복문

```python
def main():
    cnt = 0 # cnt 변수 초기화 필요
    while cnt < 3:
        print(cnt, end = ' ')
        cnt += 1
main() # 결과: 0 1 2

def main():
    i = 0
    while i < 100:
        print(i, end = ' ')
        i += 1
        if i == 20:
            break # i가 20이 되면 반복문 종료
main() # 결과: 0 1 2 ... 19
```
<br><br>

### continue 문

```python
for i in range(1, 11):
    if i % 2 == 0: # 짝수이면
        continue   # 현재 반복 건너뛰고 다음 반복으로
    print(i, end = ' ') # 결과: 1 3 5 7 9
```
<br><br>

## 3. print 함수

### print의 매개변수

- `end`: 각 출력 항목의 끝에 추가될 문자열을 지정합니다. 기본값은 개행 문자(`\n`)입니다.
- `sep`: 여러 항목을 출력할 때 항목들 사이에 추가될 문자열을 지정합니다. 기본값은 공백입니다.

```python
for i in [1, 2, 3]:
    print(i, end = '_') # 결과: 1_2_3_

for i in [1, 2, 3]:
    print(i, end = ' ') # 결과: 1 2 3

print(1, 2, 3, end = ' m^^m ') # 결과: 1 2 3 m^^m

print(1, 2, 3, sep = ', ') # 결과: 1, 2, 3

print(1, 2, 3, sep = ' _ ', end = ' m^^m ') # 결과: 1 _ 2 _ 3 m^^m
```
<br><br>

## 4. 리스트 (List)

### 리스트는 여러 값을 담을 수 있는 **변경 가능한(mutable)** 시퀀스 자료형입니다.
<br><br>

### 리스트 생성 및 연산

```python
my_list = [1, 2, 3]
nested_list = [1, 2, [3, 4], ['AAA', 'ZZZ']]

# 리스트 연결
[1, 2, 3] + [4, 5] # 결과: [1, 2, 3, 4, 5]

# 리스트 반복
[1, 2, 3] * 2 # 결과: [1, 2, 3, 1, 2, 3]
```
<br><br>

### 인덱싱 및 슬라이싱

- `[]`: 인덱싱 연산 (하나의 요소에 접근)
- `[:]`: 슬라이싱 연산 (부분 리스트에 접근)

```python
st = [1, 2, 3, 4, 5, 6, 7]

st[0] # 결과: 1 (첫 번째 요소)
st[4] # 결과: 5 (다섯 번째 요소)

st[2 : 5] # 결과: [3, 4, 5] (인덱스 2부터 4까지)
st[ : 3] # 결과: [1, 2, 3] (처음부터 인덱스 2까지)
st[2 : ] # 결과: [3, 4, 5, 6, 7] (인덱스 2부터 끝까지)
st[ : ] # 결과: [1, 2, 3, 4, 5, 6, 7] (리스트 전체 복사)

# 슬라이싱을 이용한 리스트 변경/삭제
st = [1, 2, 3, 4, 5, 6, 7]
st[ : ] = [0] # 리스트 전체를 0 하나로 교체
print(st) # 결과: [0]

st = [] # 텅 빈 리스트 생성

st = [1, 2, 3]
st[ : ] = [] # 리스트 내용 전체 삭제
print(st) # 결과: []

# 슬라이싱을 이용한 간격 지정
st = [1, 2, 3, 4, 5, 6, 7]
st[0 : 8 : 2] # 결과: [1, 3, 5, 7] (0부터 7까지 2칸 간격)
st[0 : 8 : 3] # 결과: [1, 4, 7] (0부터 7까지 3칸 간격)
```
<br><br>

### 리스트 함수

| 함수/메서드 | 설명 | 예시 | 결과 |
| --- | --- | --- | --- |
| `len(s)` | 리스트 `s`의 길이(요소 개수)를 반환 | `len([1, 2, 3])` | `3` |
| `min(s)` | 리스트 `s` 중 가장 작은 값을 반환 | `min([1, 2, 3])` | `1` |
| `max(s)` | 리스트 `s` 중 가장 큰 값을 반환 | `max([1, 2, 3])` | `3` |
| `st.append(x)` | 리스트 `st`의 끝에 `x`를 추가 | `st=[1].append(2)` | `st=[1, 2]` |
| `st.clear()` | 리스트 `st`의 모든 내용 삭제 | `st=[1,2].clear()` | `st=[]` |
| `st.insert(i, x)` | 리스트 `st`의 인덱스 `i`에 `x`를 삽입 | `st=[1,2].insert(1,3)` | `st=[1, 3, 2]` |
| `st.pop(i)` | 리스트 `st`의 인덱스 `i`의 요소를 반환하고 삭제 | `st=[1,2].pop(0)` | `1, st=[2]` |
| `st.remove(x)` | 리스트 `st`에서 첫 번째로 발견되는 `x`를 삭제 | `st=[1,2,1].remove(1)` | `st=[2, 1]` |
| `st.count(x)` | 리스트 `st`에 등장하는 `x`의 개수 반환 | `[1,2,1].count(1)` | `2` |
| `st.index(x)` | 리스트 `st`에 처음 등장하는 `x`의 인덱스 반환 | `[1,2,3].index(2)` | `1` |
<br><br>

### del 문을 이용한 삭제

```python
st = [1, 2, 3, 4, 5]
del st[ : ] # 리스트에 저장된 값 모두 삭제
print(st) # 결과: []

st = [1, 2, 3, 4, 5]
del st[3 : ] # st[3] 부터 그 뒤까지 모두 삭제
print(st) # 결과: [1, 2, 3]

st = [1, 2, 3]
del st # 리스트 통째로 삭제 (변수 st 자체가 사라짐)
# print(st) # NameError: name 'st' is not defined
```
<br><br>

## 5. 문자열 (String)

### 문자열은 여러 문자를 담을 수 있는 **수정 불가능한(immutable)** 시퀀스 자료형입니다.
<br><br>

### 문자열 함수 및 메서드

| 함수/메서드 | 설명 | 예시 | 결과 |
| --- | --- | --- | --- |
| `len(s)` | 문자열 `s`의 길이를 반환 | `len("Hello")` | `5` |
| `min(s)` | 문자열 `s` 중 가장 작은 문자(ASCII 값 기준) 반환 | `min("abc")` | `'a'` |
| `max(s)` | 문자열 `s` 중 가장 큰 문자(ASCII 값 기준) 반환 | `max("abc")` | `'c'` |
| `s.count(sub)` | 문자열 `s`에 `sub`가 등장하는 횟수 반환 | `"banana".count("na")` | `2` |
| `s.lower()` | 문자열 `s`의 내용을 전부 소문자로 바꾼 문자열 반환 | `"Hello".lower()` | `'hello'` |
| `s.upper()` | 문자열 `s`의 내용을 전부 대문자로 바꾼 문자열 반환 | `"Hello".upper()` | `'HELLO'` |
| `s.lstrip()` | 문자열 `s`의 앞에 위치한 공백을 모두 제거한 문자열 반환 | `"  Hello ".lstrip()` | `'Hello '` |
| `s.rstrip()` | 문자열 `s`의 뒤에 위치한 공백을 모두 제거한 문자열 반환 | `"  Hello ".rstrip()` | `'  Hello'` |
| `s.strip()` | 문자열 `s`의 앞과 뒤에 위치한 공백을 모두 제거한 문자열 반환 | `"  Hello ".strip()` | `'Hello'` |
| `s.replace(old, new)` | 문자열 `s`의 `old`를 `new`로 교체한 문자열 반환 | `"Hello".replace("o","a")` | `'Hella'` |
| `s.replace(old, new, count)` | `count`만큼만 `old`를 `new`로 교체 | `"ooxxoo".replace("oo","ee",1)` | `'eexxoo'` |
| `s.split()` | 문자열 `s`를 공백을 기준으로 나눠 리스트에 담아 반환 | `"Hello World".split()` | `['Hello', 'World']` |
| `s.split(x)` | `x`를 기준으로 문자열을 쪼개서 리스트에 담아 반환 | `"apple,banana".split(',')` | `['apple', 'banana']` |
<br><br>

### 문자열 탐색 관련 함수들

| 메서드 | 설명 | 예시 | 결과 |
| --- | --- | --- | --- |
| `s.find(sub)` | 문자열 `s`에 `sub`가 있으면, 그 위치의 인덱스 값 반환. 없으면 `-1` 반환 | `"hello".find("ell")` | `1` |
| `s.rfind(sub)` | `s.find()`는 앞에서부터 찾지만, `s.rfind()`는 뒤에서부터 찾음. | `"ababa".rfind("aba")` | `2` |
<br><br>

### `True` 또는 `False`를 반환하는 함수들

| 메서드 | 설명 | 예시 | 결과 |
| --- | --- | --- | --- |
| `s.isdigit()` | 문자열 `s`가 숫자로만 이뤄져 있으면 `True`, 아니면 `False` 반환 | `"123".isdigit()` | `True` |
| `s.isalpha()` | 문자열 `s`가 알파벳으로만 이뤄져 있으면 `True`, 아니면 `False` 반환 | `"abc".isalpha()` | `True` |
| `s.startswith(prefix)` | 문자열 `s`가 `prefix`로 시작하면 `True`, 아니면 `False` 반환 | `"hello".startswith("he")` | `True` |
| `s.endswith(suffix)` | 문자열 `s`가 `suffix`로 끝나면 `True`, 아니면 `False` 반환 | `"hello".endswith("lo")` | `True` |
<br><br>

```python
#is_phone_num.py
def main():
			pnum = input("스마트폰 번호 입력: ")
			if pnum.isdigit() and pnum.startswith("010"):
						print("정상적인 입력입니다.")
			else:
						print("정상적이지 않은 입력입니다.")
						
main()
```
<br><br>

## 6. 조건문 (`if`, `elif`, `else`)

```python
def main():
			num = int(input("정수 입력"))
			
			if num > 0:
					print("0 보다 큰 수입니다.")
			elif num < 0:
					print("0 보다 작은 수 입니다.")
			else:
					print("0 으로 판단이 됩니다.")
					
main()
```
<br><br>

## 7. `in`, `not in` 연산자

```python
s = "hello"

# find()를 이용한 방법
if s.find("ghe") != -1:
    print("있습니다.")
else:
    print("없습니다.")

# in 연산자를 이용한 방법 (동일한 기능, 더 파이썬스러운 표현)
if "ghe" in s:
    print("있습니다.")
else:
    print("없습니다.")
```
<br><br>

```python
print(3 in [1, 2, 3]) # 결과: True
print(4 in [1, 2, 3]) # 결과: False

print("he" not in "hello") # 결과: False
print("oo" not in "hello") # 결과: True
```
<br><br>

### 논리 연산자

- `and`: 두 조건 모두 `True`일 때 `True`
- `or`: 두 조건 중 하나라도 `True`일 때 `True`
- `not`: 조건을 반전
- `in`, `not in`: 포함 여부 확인

**참고:** 파이썬에서 `0`은 `False`로 간주되며, `0` 이외의 모든 숫자는 `True`로 간주됩니다.
<br><br>

## 8. 튜플 (Tuple)

### 튜플은 여러 값을 담을 수 있는 **값 수정 불가능한(immutable)** 시퀀스 자료형입니다.

```python
tpl = (1, 2, 3)
type(tpl) # 결과: <class 'tuple'>
```
<br><br>

### 튜플을 사용하는 이유

리스트와 달리 튜플은 한 번 생성되면 내용을 변경할 수 없으므로, 데이터의 안정성을 확보해야 할 때 유용합니다. 예를 들어, 변경되어서는 안 되는 기록(생년월일 등)을 저장할 때 사용합니다.

```python
# 리스트를 통해 관리 (정보 출력이 힘들다.)
frns = ['동수', 131120, '진우', 130312, '선영', 130904]

# 리스트 안에 리스트 (수정 가능)
frns_list = [['동수', 131120], ['진우', 130312], ['선영', 130904]]
print(frns_list[1]) # 결과: ['진우', 130312]

# 리스트 안에 튜플 (튜플 부분은 수정 불가)
frns_tuple = [('동수', 131120), ('진우', 130312), ('선영', 130904)]
print(frns_tuple[2]) # 결과: ('선영', 130904)
print(frns_tuple[1][1]) # 결과: 130312
print(frns_tuple[1][0]) # 결과: '진우'

# frns_tuple[1][1] = 130313 # TypeError: 'tuple' object does not support item assignment
```
<br><br>

리스트는 수정이 가능함을 학습했다. 이는 진우의 생년월일 정보를 누군가의 실수로 언제든지 바꿀 수 있다는 것이다. 따라서 위와 같이 튜플로 묶어서 코드에 안정성을 부여하는 것이 좋다.
<br><br>

### 튜플 관련 함수와 연산들

| 함수/연산 | 설명 | 예시 | 결과 |
| --- | --- | --- | --- |
| `len(t)` | 튜플 `t`의 길이(요소 개수)를 반환 | `len((1, 2, 3))` | `3` |
| `max(t)` | 튜플 `t` 중 가장 큰 값을 반환 | `max((1, 2, 3))` | `3` |
| `min(t)` | 튜플 `t` 중 가장 작은 값을 반환 | `min((1, 2, 3))` | `1` |
| `t.count(x)` | 튜플 `t`에 등장하는 `x`의 개수 반환 | `(1,2,3,1,2).count(2)` | `2` |
| `t.index(x)` | 튜플 `t`에 처음 등장하는 `x`의 인덱스 반환 | `(1,2,3).index(1)` | `0` |
| `x in t` | `x`가 튜플 `t`에 포함되어 있는지 확인 | `3 in (1, 2, 3)` | `True` |
| `x not in t` | `x`가 튜플 `t`에 포함되어 있지 않은지 확인 | `2 not in (1, 2, 3)` | `False` |
| `t1 + t2` | 튜플 `t1`과 `t2`를 연결 | `(1,2)+(4,5)` | `(1, 2, 4, 5)` |
| `t * n` | 튜플 `t`를 `n`번 반복 | `(1,2)*2` | `(1, 2, 1, 2)` |
| `t[start:end]` | 슬라이싱 연산 | `(1,2,3,4)[0:3]` | `(1, 2, 3)` |
<br><br>

## 9. 레인지 (range)

### `range`는 숫자의 시퀀스를 생성하는 변경 불가능한(immutable) 시퀀스 타입입니다.

주로 `for` 반복문과 함께 사용됩니다.

```python
r = range(1, 10) # 1부터 9까지의 범위
type(r) # 결과: <class 'range'>
```
<br><br>

### `list()`/`tuple()`로의 변환

```python
list(range(1, 10)) # 결과: [1, 2, 3, 4, 5, 6, 7, 8, 9]
tuple(range(1, 10)) # 결과: (1, 2, 3, 4, 5, 6, 7, 8, 9)

# 다른 시퀀스 타입도 list/tuple로 변환 가능
list("Hello") # 결과: ['H', 'e', 'l', 'l', 'o']
tuple("Hello") # 결과: ('H', 'e', 'l', 'l', 'o')
```
<br><br>

### 다양한 range 사용법

`range(start, stop, step)` 형태로 사용하며, `step`은 증가/감소폭을 나타냅니다.

```python
list(range(1, 10, 2)) # 1부터 9까지 2씩 증가: [1, 3, 5, 7, 9]
list(range(1, 10, 3)) # 1부터 9까지 3씩 증가: [1, 4, 7]

list(range(10, 2)) # 시작이 끝보다 크고 step이 양수이면 빈 리스트: []
list(range(10, 2, 1)) # 시작이 끝보다 크고 step이 양수이면 빈 리스트: []

list(range(10, 2, -1)) # 10부터 3까지 1씩 감소: [10, 9, 8, 7, 6, 5, 4, 3]
list(range(10, 2, -2)) # 10부터 4까지 2씩 감소: [10, 8, 6, 4]
```