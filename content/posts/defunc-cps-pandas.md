+++
title = 'Дефункционализация, CPS и Pandas'
date = 2024-04-13T20:04:53+03:00
+++

## Использование apply в питоновской библиотеке Pandas

Типичное преобразование данных при работе с `Pandas` зачастую выглядит следующим образом:

```python
import pandas as pd

df = pd.DataFrame({
    'City': ['Moscow', 'Stavropol', 'Volgograd'],
    'Temperature (C)': [22, 28, 18]
})

def celsius_to_fahrenheit(c):
    return (c * 9/5) + 32

df['Temperature (F)'] = df['Temperature (C)'].apply(celsius_to_fahrenheit)
```

В целом, чаще всего мы просто ходим совершить некоторое преобразование данных в столбце (или создать новый) при помощи
какой-либо нашей функции, которая отражает ту или иную прикладную логику.

Зачастую, нам нужно соверишть множество таких преобразований, к счастью, `Pandas` позволяет нам "чейнить" применения `apply` в цепочки
для проведения серии трансформации значений.

Функция `apply` выступает здесь как функция высшего порядка, принимая другую функцию, которая будет применена к каждому значению в серии.
Однако последовательное применение множества `apply` к значениям может иметь некоторые недостатки.

Рассмотрим один из способов улучшить этот процесс:

## Дефункционализация

[Дефункционализация](https://en.wikipedia.org/wiki/Defunctionalization) - это техника из функционального программирования, когда мы заменяем в функции высшего порядока 
агрумент, который является функцией, на пользовательский тип данных, в который мы включаем все желаемые операции над данными.
Таким образом, мы превращаем функцию высшего порядка в функцию первого порядка.

Простейший пример:

```python
def process_numbers(nums, operation):
    return [operation(num) for num in nums]

def add_one(x):
    return x + 1

def square(x):
    return x * x

# Пример использования
nums = [1, 2, 3, 4]
processed_nums_add = process_numbers(nums, add_one)   # [2, 3, 4, 5]
processed_nums_square = process_numbers(nums, square) # [1, 4, 9, 16]
```

Мы можем преобразовать в:

```python
from enum import Enum, auto

class Operation(Enum):
    ADD_ONE = auto()
    SQUARE = auto()

def apply_operation(x, op):
    if op == Operation.ADD_ONE:
        return x + 1
    elif op == Operation.SQUARE:
        return x * x
    else:
        raise ValueError("Unsupported operation")

```python
def process_numbers_defun(nums, op):
    return [apply_operation(num, op) for num in nums]

# Пример использования
nums = [1, 2, 3, 4]
processed_nums_add = process_numbers_defun(nums, Operation.ADD_ONE)   # [2, 3, 4, 5]
processed_nums_square = process_numbers_defun(nums, Operation.SQUARE) # [1, 4, 9, 16]
```

Второй вариант выглядит гораздо более громоздким, к тому же нам надо заранее выделить все потенциальные операции над данными,
которые мы хотим использовать. И далее такой код получается довольно "закрытым" для расширения.
С другой стороны, мы получаем взамен ограниченный и безопасный для работы с данными тип, своего рода микрофреймворк, который
прозрачно и ясно дает понять, какие именно манипуляции с данными соверщаются, запрещая при этом произвольные преобразования.
Кроме того, второй вариант легко сериализовать для сохранения или передачи.

Используя эту технику, мы можем прийти к следующиму коду, который позволяет применить некоторые операции к цене условного продукта.
Здесь класс `ColumnOP` использует метод `apply`, в котором содержатся заранее прописанные преобразования над данными.

```python
import pandas as pd
import math

class ColumnOP:
    def __init__(self, operation, value=None):
        self.operation = operation
        self.value = value

    def apply(self, series):
        if self.operation == 'Get discount':
            return value - series 
        elif self.operation == 'Get discount percent':
            return round(value - series) // value), 2)
        elif self.operation == 'Round up':
            return math.ceil(series)
        else:
            raise ValueError("Unsupported operation")

discount20_op         = ColumnOperation('Get discount', 20)
discount_percent20_op = ColumnOperation('Get discount percent', 20)
ceil_op               = ColumnOperation('Round up')

df_sales['Discount']         = discount20_op.apply(df['Sales'])
df_sales['Disrount Percent'] = discount_percent20_op.apply(df['Sales'])
df_sales['Price Rounded']    = ceil_op.apply(df['Product Price'])
```

Полученный нами код выглядит не слишком удобно при практическом использовании.
Если мы хотим выполнить серию преобразований, нам приходится оборачивать новое значение в следующее.
Если у нас есть устоявшиеся серии преобразований, было бы очень удобно выстраивать их в пайп,
схоже с тем, как мы обычно можем чейнить методы в `Pandas`.

## Continuation-passing style

Здесь нам может помочь так называемый [`Continuation-passing style`](https://en.wikipedia.org/wiki/Continuation-passing_style),
который в функциональном программировании зачастую используется как раз совместно с дефункциализацией.

Его идея в следующем: для функций, которые преобразовывают данные мы вводим дополнительный аргумент, который как раз принимает
функцию-продолжение, которая говорит, что делать дальше с полученным результатом. Эта же функция в свою очередь вызывает следующее продолжение и т.д.
пока мы не придем к какому-либо логическому завершению. Такой стиль позволяет нам выстраивать большие цепоки преобразования данных, которые 
больше не "возвращают" ни на каком этапе промежуточные результаты (зато позволяют при прерывании операций снова воспроизвести продолжение, так как содержат
информацию о последней выполненной задаче). В конце такое преобразование заканчивается желаемым побочным эффектом, например записью в файл.

Простой пример:

```python
def add_cps(x, y, continuation):
    result = x + y
    continuation(result)

# Функция-продолжение, которая просо печатает результат
def print_result(result):
    print(f"The result is: {result}")

# Используем CPS функцию
add_cps(3, 4, print_result)
```

Попробуем применить идею CPS, чтобы было возможно "продолжать" каждую операцию над столбцом следующей.
При этом конечная цель в цепочке продолжения всегда схожая - это может быть вывести полученную серию на экран, или, 
скажем, сохранить её в файл.

Немного изменим синтаксис - теперь класс `ColumnOP` получает в качестве еще одного агрумента продолжение:

```python
import math
from typing import Self, Any

import pandas as pd


df = pd.DataFrame({'Product Price': [40,52,69]}) 


class ColumnOP:
    def __init__(self, 
                 operation: str, 
                 value: dict[str, Any] | None = None,
                 next_op: Self | None = None):
        self.operation = operation
        self.value = value
        self.next_op = next_op

    def apply(self, series: pd.Series):
        match self.operation:
            case 'Apply discount': return self.next_op.apply(series - self.value)
            case 'Apply discount percent': return self.next_op.apply(round(series * (100 - self.value / 100)), 2))
            case 'Round up': return self.next_op.apply(series.apply(math.ceil))
            case 'Print Results': print(series)
            case _: raise ValueError("Unsupported operation")
```

Полученным нами код выглядит очень элегантно и наглядно показывает, какие именно преобразования с данными мы совершаем.

```python
discount_1_op = ColumnOP('Apply discount',         value=1,
      next_op = ColumnOP('Apply discount percent', value=20,
      next_op = ColumnOP('Round up',
      next_op = ColumnOP('Print Results'))),
)

discount_1_op.apply(df['Product Price'])
```

## Заключение

Сочетание CPS с дефункционализаей - это один из широко используемых в функциональном программировании приёмов, которая 
широко используется в программной инженерии, однако не слишком известна в мейнстриме. Надеюсь рассмотренный пример
наглядно показывает, что в ФП есть множество довольно универсальных и простых техник, которые вполне могут найти
применение, особенно при работе с данными.

