+++
title = 'F-ограниченный полиморфизм на Python'
date = 2024-06-04T21:39:37+03:00
+++

Параметрический полиморфизм - это свойство языка, которое позволяет в некоторых ситуациях
выполнять один и тот же код для разных типов.
В `Python` 3.12 ввели гораздо ввели гораздо более удобный синтаксис для выражения параметрического полифорфизма (т.е. удобную форму для дженериков).

```python
class Queue[T]:
    def __init__(self) -> None:
        self.elements: deque[T] = deque()

    def push(self, element: T) -> None:
        self.elements.append(element)
```

Последние годы `Python` благодаря развитию синтаксиса для аннотирования и библиотеки `typing`, а также инструментов вроде
[mypy](https://github.com/python/mypy) и [pyright](https://github.com/microsoft/pyright) стал позволять выражать очень много идей,
которые раньше находили применение только в языках с строгой типизацией, вроде `Java` и `C++`.

Одная из таких идей как раз связана с параметрическим полиморфизмом, это так называемый F-ограниченный полиморфизм. 
Что он из себя представляет (далее идёт моя попытка объяснить, в первую очередь себе, как это и зачем)?

-  Полиморфизм: позволяет использовать разные типы в рамках одного и того же кода. 
Это одна из ключевых особенностей в ООП, позволяющая писать более гибкий код.
-  Ограниченный полиморфизм: это способ ограничить типы, которые можно использовать в дженериках.
Например, вы можете сказать, что параметр типа может быть только таким, который наследует определенный класс или интерфейс.
-  F-ограниченный полиморфизм: это более конкретная форма ограниченного полиморфизма, 
при которой параметр типа не просто ограничен родительским классом или интерфейсом, но ограничен таким образом, что рекурсивно включает в себя сам тип.

Зачем использовать F-ограниченный полиморфизм?

Он позволяет нам возвращать тот же самый тип, что и объект к которому он был применен.
Если всё еще не понятно, то попробуем сконструировать пример на `Python`, который позволил бы нам выразить эту идею:

```python
from typing import TypeVar, Protocol, override

# Определим переменную типа-генерика, которая связана с протоколом "Comparable"
T = TypeVar('T', bound='Comparable', contravariant=True)


class Comparable(Protocol[T]):
    def compare_to(self: T, other: T) -> int: ...


# Класс, который поддерживает протокол "Сравнения"
class Record(Comparable['Record']):
    def __init__(self, name: str, length: int):
        self.name = name
        self.length = length

    def __repr__(self):
        return f'Record(length={self.length})'

    @override
    def compare_to(self, other: 'Record') -> int:
        return int(self.length > other.length)


# Пример
record1 = Record("Sales 2022", 2023)
record2 = Record("Sales 2023", 2024)

print(record1.compare_to(record2))  # Результат: -1 потому что 2023 < 2024
print(record2.compare_to(record1))  # Результат:  1 потому что 2024 > 2023
```

- Протокол `Comparable` специфицирует метод сравнения (`compare_to`).
- Класс `Record` реализует протокол сравнения, при этом мы специфично указываем, что сравнение подразумевается с другим объектом `Record`.
- Таким образом, метод `compare_to` как раз сравнивает 2 экземпляра класса `Record`.

При этом другие классы также могут реализовывать этот протокол, подразумевая, что сравнение происходит только между их объектами.
При "некорректном" использовании, `pyright` явно укажет нам на ошибку, если мы пытаемся сравнивать объекты разных классов.

Хотя дженерики и не "встроены" в язык `Python`, использование подсказок типов и протоколов из модуля `typing` позволяет добиться похожего результата, 
делая код более типобезопасным.

Как мы видим, на сегодняшний день `Python` позвояет разработчику писать при желании очень выразительный код при помощи аннотирования и проверкой
при помощи линтеров, при этом не обязывая его явно указывать типы там где это излишне (хотя нужно отметить, что черезмерное аннотирование часто приводит
к очень уродливому коду).
