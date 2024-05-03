+++
title = 'Как спроектировать долгосрочное API.'
date = 2024-05-02T20:49:58+03:00
+++

Многие современные API сужествуют годами и вынуждены поддерживать обратную совместимость на протяжении десятков версий.
Одна из самых известных библиотек для линейной алгебры и прочих математических вычислений - NumPy сужествует с 2005 года
и с тех пор поддерживает постоянство в дизайне API.

При планировании интерфейса программ нужно держать в голове то, что если на него будет завязано множество внешних систем,
возможно его уже никогда не получится изменить. Поэтому дизайн API так важен - большинство проблем должны быть решены на самом раннем этапе,
при этом подразумевая всевозможные изменения, которые могут произойти в будущем.

Рассмотрим следующий код, который используется для формирования некоторых отчетов:

```python
class ReportConfig(BaseModel):
    start_date: str
    end_date: str
    emails: list
    user: str
    config: Union[dict, None] = None


@router.post('/v1/send_report')
async def send_report(report_config: ReportConfig, db: Session = Depends(get_db)):
    """
    Формирование отчетов и отправка их на адрес
    """
    log.debug(f'Получены данные: {report_config}')

    report_data = {'report_data': {'emails': report_config.emails,
                                   'start_date': report_config.start_date,
                                   'end_date': report_config.end_date,
                                   'user': report_config.user,
                                   'config': report_config.config},
                   'status': 'created'}

    report_result = db.execute(insert_report(report_data)).all()

    post_msg_to_queue(emails=report_config.emails,
                      start_date=report_config.start_date,
                      end_date=report_config.end_date,
                      user=report_config.user,
                      report_id=report_result[0][0]
                      config=report_config.config)

    return JSONResponse(content={'status': 200, 'message': 'Отчеты отправлены на формирование'},
                        status_code=200)
```

Подразумевая, что наш API должен быть устойчивым, какие узязвимости есть на `/v1/send_report`?
Что если в `ReportConfig` поменяется, например, набор полей? Наша система уязыима к этому изменению,
но нет ли способа показать это более наглядно?

Есть способ:

Предполагая изменения в `ReportConfig`, попробуем протестировать поведение API. Для этого напишем простой декоратор:

```python
def remove_attribute(attribute_names: list[str]):
    def decorator(cls: Type[BaseModel]):
        for attribute_name in attribute_names:
            if hasattr(cls, attribute_name):
                delattr(cls, attribute_name)
        return cls
    return decorator
```

Такой декоратор позволит произвольно удалить атрибут модели данных.

Используем для тестирования (в продовой версии обойдемся, естественно, не будем указывать зачения в `remove_attribute`).

```python
# @remove_attribute([]) для прода будет следующий декоратор
@remove_attribute(['start_date'])
class ReportConfig(BaseModel):
    start_date: str
    end_date: str
    emails: list
    user: str
    config: Union[dict, None] = None
```

Теперь, если мы используем код для `send_report` выше, у нас возникнут проблемы на строчке с `report_data`.
Поэтому в `send_report` лучше использовать другой подход, например, итерироваться по `ReportConfig.__fields__` чтобы проходится по его полям.

Такой приём очень наглядно показывает места в прогамме, где мы слишком полагаемся на "хард-код". Такие места и будут 
первой причиной неустойчивости нашего API.

При планировании дизайна API полезно для тестирования "нагружать" его такими изменениями, чтобы лучше предсказать его поведение в будущем и
принять меры заранее.
