+++
title = 'Об информационной избыточности'
date = 2024-05-13T21:52:43+03:00
+++

Много лет назад ключевым походохом к хранению информации была эффективность с точки зрения оптимизации хранимого объема. 
Нормализованные модели данных позволяют хранить данные в максимально эффективном виде,
также обновлять и поддерживать консистентность данных однако у этого подхода есть недостатки:
если наша модель достаточно сложная, на получение данных для визуализации могут понадобиться десятки JOIN'ов

В современном мире ситуация изменилась:

- Обычно цена за дополнительное место на сервере невелика.
- Создание и поддержание дополнительных информационных моделей может быть затруднительно, но не настолько затруднительно,
как например улучшение пользовательского опыта, когда ему приходится ждать лишее время на подгрузку данных.
- OLAP базы данных дают широкий простор для организации одной и той же информации в разном представлении.

Очень часто гораздо лучше хранить больше информации, расплачиваясь местом на диске, и необходимостью иногда поддерживать расширенную модель данных.
Такйо подход даёт нам больше свободы в том, как "доставлять" данные до конечного потребителя, ведь нам больше не нужно думать,
сколько времени это займет, если эти данные будут подготовлены заранее.

Например, "широкие" пред-агрегированные таблицы позволяют избежать JOIN'ов и получать данные "напрямую".
Такие БД как `Clickhouse`, например, гораздо больше "заточены" под эту идею, и получение данных будет работать ощутимо быстрее.
Например, если мы хотим визуализировать продажи продуктов по площадкам и продажи продуктов по регионам, то есть вероятность,
что нам не стоит "собирать" эту информацию каждый раз, выбирая поле для GROUP BY (регион или площадка).
Вместо этого, мы можем хранить 2 представления, каждое пред-агрегированное по соответствующему полю.

`Materialized views` в `Clickhouse` созданы специально таким образом, чтобы обновлять информацию параллельно с 
вставкой в источник данных, облечая таким образом, создание представлений под разничные нужды пользователя.
Кроме того, `Clickhouse` поддерживает такие мощные инструменты, 
как [`AggregatingMergeTree`](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/aggregatingmergetree),
которые позволяют эффективно агрегировать такие данные, как средние значения.

Большинство современных БД представляют схожую возможность для создания подобных представлений, для схожих целем мы можем использовать триггеры
в `PostgreSQL`.

Идея с информационной избыточностью довольно близка мне, так подготовка и перенос данных для визуализации данных в BI-системах
как раз требует максимальной скорости передачи информации. Пользователь не будет ждать 10 минут, пока его графики обновляются.
И в таких случаях зачастую очень неэффективно хранить информацию только в каком-то одном виде и гораздо лучше подготавливать данные заранее для 
надлежащего представления в визуализации. 

Причем данные могут быть одни и те же, но формат их представления может отличаться значительно, и по рукой всегда проще иметь уже заготовленное представление.
Поэтому различного рода вью и материализованные вью, а также кэширование данных широко используются при работе с BI-системами.
При этом СУБД часто могут взять на себя роль по синхронизации данных за счет собственных механизмоов.
Ценой за удобство для пользователя будет потенциальная сложность в поддержке такой системы, с "дублированием" информации, но эта цена польностью оправдывает себя
когда речь идет о том, чтобы оперативно предоставлять данные для бизнеса.
