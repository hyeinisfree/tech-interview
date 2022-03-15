# Iterable과 Iterator
## Iterable 객체
- 반복 가능한 객체이다.
- 대표적으로 iterable한 타입 - list, dict, set, str, bytes, tuple, range
- iter()메소드에 전달될 때 Iterator를 생성한다.

## Iterator 객체

- 값을 차례대로 꺼낼 수 있는 객체이다.
- 파이썬 내장함수 `iter()` 또는 iterable 객체의 `__iter__()` 메소드로 Iterator 객체를 생성할 수 있다.
- 파이썬 내장함수 `next()` 또는 iterator 객체의 `__next__()` 메소드로 Iterable의 데이터에 순차적으로 접근할 수 있습니다.
- Iterator 객체는 항상 Iterable 객체가 된다. 하지만 Iterable 객체는 Iterator 객체가 될 수 있고 아닐 수도 있다.

ex) 리스트는 iterable 객체이지만 iterator 객체는 아니다.

```python
list = [1, 2, 3, 4, 5]

print(list) # [1, 2, 3, 4, 5]
print(next(list)) # TypeError: 'list' object is not an iterator
```

### StopIteration 예외

- 더이상 next 요소가 없으면 StopIteration Exception을 발생시킨다.

```python
list = [1, 2, 3, 4, 5]

iterator = iter(list)

print(next(iterator)) # 1
print(next(iterator)) # 2
print(next(iterator)) # 3
print(next(iterator)) # 4
print(next(iterator)) # 5
print(next(iterator)) # StopIteration
```

### Iterable 객체를 Iterator 객체로 변환

파이썬 내장함수 `iter()` 또는 iterable 객체의 `__iter__()` 메소드를 사용해 모든 Iterable 객체를 Iterator 객체로 만들 수 있다.

```python
list = [1, 2, 3, 4, 5]

iterator1 = iter(list)
iterator2 = list.__iter__() 

print(iterator1) # <list_iterator object at 0x102955fd0>
print(iterator2) # <list_iterator object at 0x102955fa0>
```

### 📗 참고
- [Python - Iterator와 Iterable의 차이점](https://codechacha.com/ko/python-difference-between-iterator-and-iterable/)