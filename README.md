# Разработка модели наукастинга для прогнозирования месячного ИПЦ на основе данных о ценах в интернет магазинах

Команда: *Young Stars*

Кейсодержатель: Центральный банк Российской
Федерации (Банк России)

## Постановка задачи

На основе открытых микроданных о потре-
бительских ценах, собранных посредством
веб-скрейпинга интернет-магазинов Мо-
сковской области (Московского региона),
создать модель наукастинга на базе мето-
дов машинного обучения, позволяющую
формировать краткосрочный (на горизон-
те 1-2 месяцев) прогноз месячного ИПЦ в
режиме реального времени.

## Описание данных

Предоставлен следующий массив данных:

| Название поля | Описание |
| --- | --- |
|WebPriceId | Уникальный номер товара/услуги|
|DateObserve | Дата наблюдения|
|StockStatus | Статус товара/услуги на дату наблюдения (InStock – в продаже, OutOfStock – отсутствует в продаже)
|CurrentPrice | Цена товара/услуги на дату наблюдения (Если StockStatus = OutOfStock – значение отсутсвует) |

## Описание решения

Мы пытались рассмотреть решение с разных сторон. 

- Сначала мы реализовали алгоритм Росстата - брали ИПЦ по товарам и потом среднее от них. MSE было 0.84, но нам было этого недостаточно.
- Мы придумали сложный пайплайн



    В задании говорится, что прогноз должен быть основан на данных за прогнозируемый месяц (т.е. если мы прогнозируем сентябрь, то учитывать данные начиная с первого сентября) и данных за весь предыдущий период.

    Вот что я предлагаю сделать:
    Чтобы учесть текущие данные за месяц мы
    1) делим на кластеры имеющиеся ряды
    2) считаем среднюю цену для кластеров в начале месяца и в день n. Делим последнее на первое - получаем ИПЦ для группы товаров. За период от 0 до дня n. Если бы хотели получить их как оценку месяца - можно домножить на 30/n.

    3) Чтобы учесть данные за прошлые месяцы - можно построить ариму по уже имеющимся значениям ИПЦ за прошлые месяцы
```
Направить вывод 2 и 3 в модель регрессии \ xgboost \ что-то другое и научить ее предсказывать на основании ИПЦ по кластерам и ожидаемому из истории ИПЦ индекс ИПЦ на конец этого месяца
```

- В итоге мы не успели реализовать этот пайплайн, но мы думаем, что он должен давать хороший результат.
- Мы использовали стохастический подход для оригинального алгоритма и улучшили MSE до 0.5 - т.е. мы делали множество предсказаний на разное количество случайно выбранных продуктов, для них считали среднее ИПЦ. Делали много таких предсказаний и в среднем результат должне отражать генеральную совокупность, из-за чего мы предполагали, что результат улучшится.
    
## Алгоритм (подробнее)
    
1. Мы отбираем данные - берем записи, релевантные для нашего периода и отбрасываем товары, у которых меньше пяти записей за этот период.
2. Выбираем k случайных товаров и считаем средний ИПЦ (выбирая краевые даты). Используется следующая формула: `ИПЦ = x[1] / x[2] * (30/interval)) - 1`, где interval - это интервал, 30/interval поправка на интервал а x1 x2 - цены в этот и предыдущий месяца соответственно. 
3. Повторить пункты 1-2 n раз
4. Из-за большого количества предсказаний, в среднем получается точный результат
5. Таким образом можно итеративно улучшать качество предсказания
    
   
## Young Stars - это мы!
- Алина Лебедева - ML специалист
- Леша Кругликов - full stack разработчик
- Мария Антонова - Дизайнер
- Михаил Усатюк - product manager
