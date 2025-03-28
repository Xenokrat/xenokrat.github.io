+++
title = 'Ещё один взгляд на TDD'
date = 2025-02-21T17:30:01+03:00
+++

Недавно в небольшом посте про [TDD]({{< ref "about-tdd" >}}) я жаловался
на то, что этот подход к написанию программ сводит на нет креативностью
в подходе к "исследованию" проблемы.

Но при этом меня не покидало ощущение, что я недопонимаю что-то важное в
подходе TDD. Мой предыдующий опыт знакомства с TDD был связан с 
"Python: Разработка на основе тестирования" Гарри Персиваля, а также нескольких
други отдельных постов о TDD.

Но недавно мне попалась книга ["Разработка через тестирование" (TDD)](https://www.piter.com/collection/biblioteka-programmista/product/ekstremalnoe-programmirovanie-razrabotka-cherez-testirovanie-2) Кента Бека,
одного из больших специалистов в области и автора (одного из) подхода "Экстремальное программирование".
И, кажется, она помогла мне взгянуть на подход под немного другим углом.

Например, при TDD тесты не обязаны быть своего рода "заранее установленной спецификацией" программы.
Даже наоборот - Кент Бек считает, что первые 2 из 3 шагов TDD - я
имею в виду цикл "красный-зелёный-рефакторинг" (пишем тест, который не проходит,
пишем код, который заставляет тест проходить, т.е. стать зелёным, затем
делаем код лучше, быстрее и т.д.) должен происходить как можно быстрее.

Допускается писать тесты, которые вероятно будут далеки от того, чтобы мы получим
как итоговый результат. Допускается писать реализацию-заглушку, которая
просто даст нам "зелёную полоску" в результате выполнения теста.

Вместо этого тест становится лишь "подпоркой". Написание кода становится похоже
на альпинистский подъём по склону - каждый шаг сопровождается созданием
подпорки (теста), которая в моменте просто даёт нам ощущение устойчивости.

Например, мы можем начать с такого теста (пример из книги):

```python
class TestMoney(TestCase):
    def test_multiplication(self):
        five = Dollar(5)
        five.times(2)
        self.assertEqual(five.amount, 10)
```

Т.е. практически "нагаллюцинировать" будущий API. Мы просто пытаемся выразить что
нам нужно, но не уславливаемся что конечная спецификация будет именно такой, или
что даже класс `Dollar` вообще будет существовать.

При этом взаимодествие с тестами постоянное. Мы часто возвращаемся к тестам,
дополняем их, изменяем, удаляем.

Очевидный минус такого подхода - это то, что на написание уходит ГОРАЗДО больше времени,
по сравнению с тем, если бы мы написали реализацию, и лишь затем приступили к тестам.
Кент Бек пишет, что важно уметь "регулировать" скорость продвижения в цикле "тест-код-рефакторинг" - почувстсвовав уверенность можно двигаться быстрее,
писать более длинные тесты и реже отвлекаться возвращением к тестам. Но базовый принцип
при этом остаётся неизменным - правила сохраняются, и мы по хорошему не должны
например писать ещё тесты, если у нас сейчас есть "красные полоски".
Поэтому даже двигаясь "большими шагами" скорость написания кода ощутимо медленнее чем
обычно, и поэтому я ощущаю себя менее продуктивным.

В целом всё еще не определился для себя, как именно я ощущаю для себя полезность
TDD.

Однако подход, описанный в "Разработка через тестирование" (TDD) по крайней мере
снял значительную часть фрустрации, которую я испытывал от подхода "сначала тест".
Дело в том, что подход "сделать небольшой шаг, затем осмотреться и оценить ситуацию"
для меня гораздо более естественен, чем заранее полностью продумывать дизайн приложения.
Но весьма вероятно, что и тут я не прав и мне просто нужно подступиться к проектированию
с другой стороны.

Интересно дальше хотя бы в кратце изучить [BDD](https://ru.wikipedia.org/wiki/BDD_(%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)) и его связь с TDD (интересно, уместен
ли BDD для соло-разработчика?).
