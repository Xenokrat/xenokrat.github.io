+++
title = 'Знакомство с Ассемблером'
date = 2024-08-23T21:31:32+03:00
+++

Отвлечёмся ненадолго от функционального программирования.

Какой самый низкий уровень, куда может "спуститься" программист? 

В современном мире у ассеблера остаётся место
для использования - обратный инжиниринг, точечная оптимизация в системном программировании (элементы операционных систем, драйвера и т.д.), хакинг.
Также это самый базовый уровень, на котором большинство программистов могут взаимодействовать с компьютером.

Кроме того, любая hardware, те же видеокарты, как правило имеет собственный ассемблер.

После нескольких дней с ассемблером, какие же у меня впечатления?

Ассемлер это вообще не весело, не смешно, не просто и не интуитивно, в отличие даже от
того же Си, который следующий в очереди по близости к реальному "железу" компьютера, и который в сравнении
кажется образцом "человеческого" языка.

Дело в том, что "базово" понять концепцию не так уж сложно:
Есть команды - инструкции заложенные в процессор изначально.
Есть регистры - ячейки быстрой памяти процессора, системные вызовы функций ОС, 
лейблы и jump-команды, которые осуществляют ветвление и циклы.
Не все операции можно сделать напрямую, для этого существует `syscall`, системный
вызов, это способ "попросить" операционную систему сделать что-то - прочитать stdin, вывести
в stdout, открыть файл и т.д.- 

[Ресурс по syscall'ам в Linux](https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md)

Если разобраться в том, как создаётся стек вызова функций и при чем тут стек-поинтеры, то
можно писать "серьезные" программы.
В конце концов, всё можно свести к последовательному набору инструкций, который иногда прерывается прыжком к лейблу,
но вся сложность здесь в постоянно меняющемся состоянии.
На практике всё это требует огромной концентрации внимания и понимания когда и как использовать
конкретный регистр, какие инструкции и системные вызовы изменяют другие регистры, что такое PIE и как работает адресация.
(Eщё не затрагиваю тот факт, что инструкций для процессора x86 более 4000, на практике конечно в 90% случаев используется около пары десятка.
Но людям, которые пишут компиляторы видимо действительно приходится их изучать :)).

Да, дебаггер `gdb` это единственный способ примерно понять, что там происходит.
Ну и в довесок, ассемблер полностью лишен концепции "обработки исключений". Если программа на 200+ строк где-то работает
не так, то удачи разобраться (даже debug-print просто не вставить).

У меня ушло несколько часов на то, чтобы написать простую утилиту `fold`, которая каждый раз переносит текст на новую
строку по указанному пользователем лимиту. Большую часть этих часов я смотрел как:

- Программа segfault'иться
- Программа не выводит ничего
- Программа пишет невразумительную ошибку о том, что мой код не `Position-independent`.

Умение писать ассемблер видимо всё-таки требует хорошего понимания работы процессоров и как устроены и выполняются инструкции для него.
После этого у меня осталось только безмерное уважение к людям, которые профессионально пишут на ассамблере хоть что-то.

На случай если кто-то пропустил: [статья](https://habr.com/en/articles/445918/) про Криса Сойера, который написал 
полноценную игру **RollerCoaster Tycoon**, почти полностью **ассемблере x86!!!**.

Да, интересный факт: ChatGTP-4o вообще не может писать ассемблер сложнее "Hello, World" ни в каком виде,
по крайней мере с моими попытками писать NASM. ИИ постоянно путается в регистрах, мгновенно забывает
какие значения туда положил, галлюцинирует несуществующие адреса, и мне не удалось добиться от него даже небольших осмысленных участков кода.

Стратегия гораздо лучше - проект [Compiler Explorer](https://godbolt.org/), где можно писать небольшие кусочки кода на Си (или другом компилируемом языке, выбор довольно большой)
и смотреть на полученный ассемблер.

Интересный факт номер два: по-видимому, даже написанный работающий ассемблер - это не всегда и не совсем те инструкции, которые выполняет процессор в итоге.
Еще существует микрокод процессора, который может вносить свои корректировки. Так что даже здесь компьютер что-то от нас скрывает :).

Думаю ли я, что базовое понимание ассемблера как-то нужно среднестатистическому программисту?
Не уверен, но что это точно даёт: взгляд на работу компьютера как есть, без прикрас и вообще каких-либо "ограждений".
И даже если вряд ли можно перенять какие-то приёмы из ассемблера для нашего любимого ЯП, такие вот принципиально
отличающиеся концепции программирования дают нам возможность какой-то другой перспективы, а это всегда не будет лишним.
