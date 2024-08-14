+++
title = 'Галопом по ФП - Моноиды'
date = 2024-08-14T09:53:07+03:00
+++

Абсолютно must-watch доклад по функциональному программированию от Скотта Влашина, владельца замечательного сайта [F# for Fun and Profit](https://fsharpforfunandprofit.com), и автора многих важных книг по функцональному программированию, вроде [# Domain Modeling Made Functional](https://pragprog.com/titles/swdddf/domain-modeling-made-functional/) (увы, еще не читал).

{{< youtube E8I19uA-wGY >}}

Вообще весь доклад - это максимально сжатое и понятное широкому кругу разработчиков введение в ФП и отличное место, чтобы примерно представить как решают проблемы там. Особенно хорошо воспринимается на контрасте с сравнением с привычными ООП подходами. 

Поэтому его здорово использовать как каркас, чтобы объяснить читателям (и себе тоже :)), важные идеи, которые можно аккуратненько "одолжить" и использовать и в других языках.

Давайте посмотрим на `моноиды`, тем более, это не самая популярная идея из функциональщины, но при этом достаточно простая.

**Моноид** - это категория объектов, но определить его не так уж сложно, даже не придется погружаться сильно в теорию категорий, достаточно 3 свойств:

1. У моноида есть операция "конкатенации", т.е. мы можем взять один моноид и "сложить" его с другим, получив при этом такой же моноид.
	1. Для для сложения целых чисел и произведения всегда получится такое же целое число.
	2. При конкатенации строк получается новая строка.
2. Ассоциативность: (1 + 2) + 3 то же самое, что 1 + (2 + 3), так же, очевидно, это работает и для строк.
3. Наличие "нейтрального" элемента: т.е. для моноидов существует такой элемент, который при "складывании" не изменяет текущий элемент: 
	1. Для сложения целых это ноль: `2 + 0 => 2 `
	2. Для произведения: `2 * 1 => 2`
	3. Для строк - пустая строка: `"hello" + "" => "hello"`
	(интересный факт, без отсутствие нейтрального элемента даёт нам так называемую `полугруппу`)

Какие плюсы может дать нам знание того, что определенный элемент является моноидом?

- Свойство 1 дает нам бинарную операцию между элементами, благодаря которой 2 моноида становятся 1. И эту операцию можно повторить сколько угодно раз, сведя набор значений к единственному. Таким образом мы получаем операцию `reduce`. 
```python
from functool import reduce

lst = [1, 2, 3]
reduce(lambda a, b: a + b, lst) # даст 6
```

- Второе свойство говорит, что порядок выполнения не важен. Поэтому моноиды так удобно использовать с алгоритмами "разделяй и властвуй". Их легко и безопасно параллелить, а также постепенно накапливать значения.
- Наконец последнее свойство позволяет задать начальное значения в цепочке, или отразить "пустое" значение.

Типичный паттерн использования моноидов подразумевает "привести" существующий объект к тому, чтобы он стал моноидом, чтобы затем использовать описанные выше свойства.

Попробуем применить эти идеи на Python: допустим у нас есть класс Customer.

```python
@dataclass 
class Customer: 
	name: str 
	age: int 
	purchase_amount: float
```

Создадим для этого класса моноид, который будет отражать статистику по приобретению товаров клиентами:

```python
@dataclass 
class CustomerStat: 
	total_customers: int 
	total_age: int 
	total_purchase: float 
	
	@staticmethod 
	def identity() -> 'CustomerStat': 
		return CustomerStat(
			total_customers=0,
			total_age=0, 
			total_purchase=0
		) 
		
	def combine(self, stat2: 'CustomerStat') -> 'CustomerStat':
		return CustomerStat(
			total_customers=self.total_customers + stat2.total_customers,
			total_age=self.total_age + stat2.total_age,
			total_purchase=self.total_purchase + stat2.total_purchase, 
		) 

	def average_age(self): 
		return (
			self.total_age / self.total_customers 
			if self.total_customers > 0 
			else 0 
		)
	
	def average_purchase(self): 
		return (
			self.total_purchase / self.total_customers 
			if self.total_customers > 0 
			else 0
		)
```

Дополнительно, нам нужна функция, которая поможет "смапить" покупателя в класс для статистики, и функция-аналог `reduce`

```python
def customer_to_stat(customer: Customer) -> CustomerStat:
    return CustomerStat(
        total_customers=1,
        total_age=customer.age,
        total_purchase=customer.purchase_amount,
    )

def aggregate_customers(customers: List[Customer]) -> CustomerStat:
    customer_stats = map(customer_to_stat, customers)
    return reduce(CustomerStat.combine, customer_stats, CustomerStat.identity())
```

Теперь мы можем быстро и легко работать с статистикой по клиентам

```python
# Пример
customers = [
    Customer(name="Alice", age=30, purchase_amount=100.0),
    Customer(name="Bob", age=25, purchase_amount=150.0),
    Customer(name="Charlie", age=35, purchase_amount=200.0)
]

aggregated_stat = aggregate_customers(customers)
print(f"Average Age: {aggregated_stat.average_age()}")
print(f"Average Purchase: {aggregated_stat.average_purchase()}")
```

Этот приём используется в функциональных языках повсеместно, но я не вижу больших препятствий для того, чтобы использовать моноиды в любых языках.
