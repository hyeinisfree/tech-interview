# Call By Assignment
파이썬의 모든 변수는 객체(Object)이다. 

파이썬은 특정 변수에 값을 할당할 때, 그 값에 해당하는 Object를 만들고 그 Object의 주소를 변수에 할당하는 방법을 사용한다. 즉, 기본적인 변수 값 전달 방법은 Pass By Address이다.

이 때 파이썬은 해당 변수가 가변(Mutable) 타입인지, 불변(Immutable) 타입인지 검사한다.
불변 객체의 경우, 파이썬은 해당 객체의 값이 연산 등의 결과로 변경되어 다른 변수에 할당 되어야 할 때, 새로운 값을 객체로 메모리에 할당한 후 그 값을 변수가 가리키게 한다.
불변 객체의 경우 주소를 전달하는 방식으로 값을 전달하면 본래 변수 값이 변할 위험이 있기 때문이다.
반면 가변 객체의 경우 주소를 전달해도 위험이 없으므로 주소로 값을 전달한다.

따라서 불변 타입의 변수는 Call By Value로 전달하는 것처럼 보이고(실제로 주소를 가르킴에도), 가변 타입 변수는 Call By Reference로 전달되는 것처럼 보인다. 이를 Call By Assignment라고 한다.


## Mutable 객체
```python
def main():
    n = 1
    print(f"Before function call, Address of n: {id(n)}")
    print(f"Before function call, Value of n: {n}")
    func(n)
    print(f"After function call, Address of n: {id(n)}")
    print(f"After function call, Value of n: {n}")
    
def func(x):
    print(f"In funtion, Address of x: {id(x)}")
    print(f"In funtion, Value of x: {x}")
    print("Change x")
    x = 10
    print(f"In funtion, Address of x: {id(x)}")
    print(f"In funtion, Value of x: {x}")
    
main()
```
<img width="404" alt="스크린샷 2022-03-15 오후 11 22 32" src="https://user-images.githubusercontent.com/46434694/158405123-74276722-6d53-42b6-b27c-ccc69856149b.png">


## Immutable 객체
```python
def main():
    n = ['Hello']
    print(f"Before function call, Address of n: {id(n)}")
    print(f"Before function call, Value of n: {n}")
    func(n)
    print(f"After function call, Address of n: {id(n)}")
    print(f"After function call, Value of n: {n}")
    
def func(x):
    print(f"In funtion, Address of x: {id(x)}")
    print(f"In funtion, Value of x: {x}")
    print("Change x")
    x.append('World')
    print(f"In funtion, Address of x: {id(x)}")
    print(f"In funtion, Value of x: {x}")
    
main()
```
<img width="404" alt="스크린샷 2022-03-15 오후 11 23 51" src="https://user-images.githubusercontent.com/46434694/158405215-4a7acfec-2411-4540-a6c6-c3087f774e7f.png">


변수를 새로 초기화할 때마다 Object를 할당한다면 나중에 쓰이지 않는 Object들이 대거 생길 것이다. 예컨대 아래와 같이 초기화를 할 경우 int 형 상수 1 Object가 메모리에 참조되지 않은 채로 남아있는 것이다.
```python
a = 1
a = 2
```
파이썬의 GC(가비지 컬렉션)가 이를 처리해준다.

만약 이전에 나온 변수와 동일한 값을 새로운 변수에 할당하려 한다면, 이전 변수에서 사용한 Object를 재활용한다. (GC에 의해 소멸되지 않았다면)
```python
a = 1
b = a + 1 - 1
print(id(a))
print(id(b))
```
<img width="189" alt="스크린샷 2022-03-15 오후 11 39 06" src="https://user-images.githubusercontent.com/46434694/158405297-b8101f93-acf8-4a49-a980-95d0b09fd97f.png">


> *id()*  
> 객체(Object)의 고유값을 리턴하는 내장 함수


### 📗 참고
* [passed by assignment](https://seung00.tistory.com/16)
