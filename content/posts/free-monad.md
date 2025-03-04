+++
title = 'Свободные Монады'
date = 2025-03-04T19:21:15+03:00
+++

Несмотря на то, что я в основном разрабатывают программы на языке Python, некоторые понятия
из функционального программирования всё равно на слуху - одно из них это монады.
Монады фундаментальны для функциональных ЯП, особенно для языков вроде `Haskell`, для которых
не допускают "не чистых" функций, поэтому монада - это единственный способ выражения и работы с состоянием.

Однако недавно я наткнулся на ещё один термин - "свободные монады".

Такие монады называются "свободными", потому что их можно собрать из любого функтора.
Но также словом "free" часто обозначается минимально приемлимое выражение какой-то концепции - т.е. "Free Monad"
это самое простое выражение концепции "монады".

Монада "Free" получается на основе какого-то фунтора, т.е. класса (типа)
который поддерживает операцию `map`. Определим такой функтор:

Попробуем реализовать хоть как-то эту концепцию на Python.

```python
from typing import TypeVar, Callable

A = TypeVar('A')
B = TypeVar('B')

class Functor[A]:
    def map(self, f: Callable[[A], B]) -> 'Functor[B]':
        raise NotImplementedError
```

Монада `Free` - это рекурсивная структура данных, которая имеет 2 случая:

- `Pure`: выражает "чистое" просто значение
- `Free`: выражение, которое производит новую Free-монаду

Также мы должны расширить список операций `flat_map` - который является
обязательным для реализации монад.
    
```python
from typing import Union

class Free[A]:
    def __init__(self, value: Union[A, Functor['Free[A]']]):
        self.value = value

    def map(self, f: Callable[[A], B]) -> 'Free[B]':
        if isinstance(self.value, Functor):
            return Free(self.value.map(lambda x: x.map(f)))
        else:
            return Free(f(self.value))

    def flat_map(self, f: Callable[[A], 'Free[B]']) -> 'Free[B]':
        if isinstance(self.value, Functor):
            return Free(self.value.map(lambda x: x.flat_map(f)))
        else:
            return f(self.value)

    @staticmethod
    def pure(a: A) -> 'Free[A]':
        return Free(a)
```

В целом, получилось похоже на выражение из Haskell (но естественно куда многословней):

```haskell
data Free f a = Pure a | Free (f (Free f a))
```

Реализуем тестовый функтор для вывода в консоль - `IO`:

```python
class IO(Functor[A]):
    def __init__(self, action: Callable[[], A]):
        self.action = action

    def map(self, f: Callable[[A], B]) -> 'IO[B]':
        return IO(lambda: f(self.action()))
```

Также нам нужна функция, которая запустит наше вычисление, назовём её `interpret`:

```python
def interpret(io: Free[A]) -> A:
    if isinstance(io.value, IO):
        return io.value.action()
    elif isinstance(io.value, Functor):
        return interpret(io.value.map(lambda x: interpret(x)))
    else:
        return io.value
```

Теперь мы можем собрать всё вместе в виде последовательных вызовов функций:

```python
# Определим несколько IO-действий
read_line: Free[str] = Free(IO(lambda: input("Enter something: ")))
write_line: Free[None] = Free(IO(lambda: print("You entered: ...")))

# "Соберём" их вместе, используя Free-монаду
program: Free[None] = read_line.flat_map(
    lambda user_input: write_line.map(lambda _: print(f"You entered: {user_input}"))
)

# Запуск программы
interpret(program)
```

Реализация вроде бы удовлетворяет требованиям монад, но тем не менее
я ещё не до конца прочувствовал смысл это концепции :).

Одно из описаний различия Free от обычных монад - это то, что тип монады
обычно позволяет выразить некоторое "вычисление" (это как раз то, как
в ФП происходит управление состоянием - работа с логами, исключениями, IO и т.д.). Это вычисление происходит в обычных монадах при "коллапсе" контекста,
т.е. контекст дополняется новой информацией. Однако с Free этого не происходит, поэтому её назначение - это просто передача с сохранением некоторого дополнительного контекста для функтора.

Возможно, лучшее объяснение вы найдете здесь: 
https://stackoverflow.com/questions/13352205/what-are-free-monads