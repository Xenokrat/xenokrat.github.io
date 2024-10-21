+++
title = 'Квантовое программирование - возможно самая странная парадигма среди ЯП'
date = 2024-10-10T22:14:17+03:00
+++

Недавно мне стало интересна технология использования квантовых вычислений, точнее, как именно
алгоритмы должны использовать особенности квантового мира для того, чтобы производить вычисления на множество порядков быстрее,
чем обычные компьютеры.

В квантовы вычисления вместо двоичной логики c 0 и 1 используются кубиты. 
Кубиты похоже на обычные биты в том смысле, что их значения также могут быть равными 0 и 1,
но особенность здесь в том, что кубит сам по себе не находится в одном из этих состояний, но пребывает в суперпозиции.
Мы не можем знать точное значение кубита без специальной операции измерения, т.к. он находится в определённом вероятностном состоянии
между 0 и 1. Эти вероятности обычно обозначаются как `A` и `B`, и означают примерно что `A^2` это вероятность перехода в состояние 0,
соответственно `B^2` - в состояние 1.
Знать заранее результат невозможно (из-за фундаментального физического закона, а не скажем нехватки технологий), а сам процесс измерения
приводит к тому, что кубит из суперпозиции переходит в одно из 2-х состояний, при этом предыдущие значения `A` и `B` теряются.
Считается, что по причине этого явления кубит (до измерения) как бы находится одновременно во всех состояниях, а значит, например,
4 кубита будут представлять собой одновременно 16 состояний.

Управлять кубитами можно, пропуская их через специальные вентили, которые изменяют их внутреннее вероятностное состояние 
(при этом сохраняется равенство `A^2 + B^2 = 1` (**условие нормализации кубита**).
Некоторые вентили могут принимать больше одного кубита - например CNOT один кубит является управляющим, и состояние второго зависит от
результата его измерения. Такие кубиты считаются "связанными", в том смысле, что информация об таких кубитах может быть получена 
из измерения другого.

Всё это только введение, и может показаться, что эта тема исключительно сложна, однако на самом деле для старта в этой теме достаточно знать на базовом уровне:

- Основы программирования на других языках (само собой :))
- Комплексные числа (для описания состояний кубитов)
- Линейная алгебра (состояния и их преобразования при помощи вентелей)
- Базовая теория вероятности.

Самый простой способ исследовать этот вопрос -- использовать симулятор квантового компьютера.
И здесь как нельзя кстати приходится чудесный open source проект Microsoft -- язык [`Q#`](https://github.com/microsoft/qsharp)
и проект [Azure Quantum Development Kit]

`Q#` очень user-friendly язык, особенно если вы уже знакомы с C#, или подобными ему языками. 
Читается очень легко и понятно.
Кроме того, всё это очень легко скачать и настроить для использования на своём компьютере,
или воспользоваться песочницей прямо в браузере.

Есть целый сервис для введения в тему квантовых алгоритмов на Q#, с решением задач:
https://quantum.microsoft.com/en-us/tools/quantum-katas

Если мы хотим использовать Q# локально в Jupyter-ноутбуках:

```sh
python -m pip install qsharp azure-quantum
```

Последний пакет нам нужен только если мы планируем использовать облачный сервис Azure.
Microsoft пытается пушить использование квантового симулятора в Azure (что включает платную подписку, хотя
там вроде дают бюджет "на попробовать"), но в целом всё хорошо работает и локально.

```sh
python -m pip install ipykernel ipympl jupyterlab
```

В целом, возможно я вернусь к этому вопросу в ближайшее время, потому что так и не приблизился
к пониманию, как конкретно должны использоваться квантовые алгоритмы, ведь даже 
если кубит находится в суперпозиции, то получить из него информацию можно только измерением,
и в конце концов мы получаем тот же самый один бит информации (но видимо они будут связаны с квантовыми
регистрами, т.е. группами связанных друг с другом кубитов).

Однако приятно, что Microsoft даёт очень хороший набор инструментов и много информации для людей, которые интересуются данной темой.