## #1 

> В БД mysql  есть одна единственная таблица «abc» с полями: id (int), name (varchar), cnt (int). В таблице содержится порядка 10 млн записей. Что нужно сделать, что быстро работали следующие запросы (3 разных случая)? Все остальные факторы кроме скорости чтения не критичны.
> - SELECT * FROM abc WHERE name = 'xxx' AND cnt = yyy
> - SELECT * FROM abc WHERE cnt = xxx AND name LIKE 'yyy%'
> - SELECT * FROM abc ORDER BY cnt ASC

`ALTER TABLE abc ADD INDEX abc_cnt_name(cnt, name);`

## #2

> У вас есть задача обработки большого объема данных в PHP (например, парсинг CSV-файлов объемом в несколько гигабайт). Как вы будете обрабатывать этот файл, чтобы избежать превышения лимита памяти, и какие функции PHP и подходы для этого ,eltnt использовать?

У бы использовал функцию-генератор. Примерно такого содержания:

```
function parseFile(string $fileName)
{
    $file = fopen($fileName, "r");
    while (!feof($file)) {
        yield fgets($file);
    }
    fclose($file);
}
```
```
foreach (parseFile('some_scv_file.csv') as $line) {
    var_dump($line);
}
```
Т.о. в памяти будет всегда только текущая строка итерации.

## #3

> Есть код и запрос к БД, чтобы вы в нем изменили? Почему?:
$DB->query("SELECT * FROM abc WHERE id=" . $_GET['id']);

Здесь нам необходимо убедится что прилетает действительно число, и обезопасить себя (от sql инъекций) приведением к `integer`.

`$DB->query("SELECT * FROM abc WHERE id=" . intval($_GET['id']));`

или возможно так:

`$DB->query("SELECT * FROM abc WHERE id=?", intval($_GET['id']));`

## #4

> В БД есть таблица заказов (orders) с полями:
> - date - дата оформления заказа
> - customer_name - имя клиента
> - order_price - сумма заказа
> 
> Напиши sql запросы для выборки:
> - Запрос, который покажет сколько денег принес каждый отдельно взятый покупатель с группировкой по месяцам.
> - Запрос, который выведет  имена клиентов, у которых суммарные покупки за весь период превысили 10 тыс. руб. и одновременно никогда не было заказов менее 500 руб.

#### Первый запрос
```
SELECT customer_name, MONTH(DATE) AS month, ROUND(SUM(order_price), 2) AS by_month_sum
FROM orders
GROUP BY month, customer_name 
ORDER BY customer_name, month;
```
#### Второй запрос
```
SELECT a.customer_name
FROM orders AS a
GROUP BY a.customer_name
HAVING SUM(order_price) > 10000 AND 
       (SELECT COUNT(*) 
        FROM orders AS b 
        WHERE b.customer_name = a.customer_name AND b.order_price < 500
       ) = 0;
```

## #5

> Есть две javascript-функции:
> `function f(a,b) { return a+b }`
> и
> `var f = function(a,b) { return a+b }`
> Есть ли между ними разница? Если есть то какая?

#### `function f(a,b) { return a+b }` обычное объявление функции

- можно вызывать до ее определения в коде
- нельзя присвоить переменной и передать другой функции
- определена в глобальной области видимости

#### `var f = function(a,b) { return a+b }` анонимная функция

- нельзя вызвать до ее определения
- можно передать другой функции как аргумент
- можно использовать внутри текущего контекста, там где определена переменная `var f`

## #6

> Чем принципиально отличаются между собой условия LEFT JOIN и INNER JOIN в sql? Какой вариант JOIN может дать потенциально больше результатов (строк) и почему?

В случае с `INNER JOIN` в результате будут только записи для которых определена связь. Если мы запросим продукты из таблицы `products` и хотим также получить название категории из связанной таблицы `categories`, то используя `INNER JOIN` мы получим только продукты для которых существуют категории.

Используя же `LEFT JOIN` в результат попадут также и продукты для которых категории не определены, и соответсвующее поле для названия категории будет содержать `NULL`.

Следовательно `LEFT JOIN` потенциально даст больший результат, по причине возвращения всех записей из левой таблицы (согласно условию), даже тех, для которых, с правой таблицы не будет соответствия.

## #7

> Вам нужно реализовать консольный php-скрипт на сервере под Unix, который бы выводил каждые 15 секунд фразу «Hello». После вывода «Hello» скрипт всегда завершает работу. Напишите этот скрипт и пошагово расскажите, что нужно сделать, чтобы выполнялись исходные условия.

Думаю эта задача для `cron`

1. Проверяем чтоб `cron` был установлен на сервере `which crontab`
2. Если не установлен, то устанавливаем `apt-get install cron`
3. Проверяем чтоб статус был "запущен" `systemctl status cron`
4. Пишем `php` скрипт `hallo.php`, например, здесь `/var/www/` следующего содержания:
```
<?php

echo 'Hallo' . PHP_EOL;
```
5. В терминале выполняем `crontab -e`
6. В конец файла добавляем строки:
```
* * * * * /usr/bin/php /var/www/hallo.php >> /var/www/hallo.log
* * * * * sleep 15; /usr/bin/php /var/www/hallo.php >> /var/www/hallo.log
* * * * * sleep 30; /usr/bin/php /var/www/hallo.php >> /var/www/hallo.log
* * * * * sleep 45; /usr/bin/php /var/www/hallo.php >> /var/www/hallo.log
```
7. В лог-файле `/var/www/hallo.log` каждые 15 секунд должна появляться строка "Hello".

## #8

> Напишите на php функцию для распределения рублевой скидки по купону на все товары в корзине пропорционально стоимости товара. На входе в функцию передаются два параметра: размер скидки в рублях (!) и массив из цен товаров, на выходе тот же массив цен, но уже с учетом скидки: distribute_discount(int $discount, array $prices) → return array $prices;

```
function distribute_discount(int $discount, array $prices): array
{
    if (empty($prices)) {
        return [];
    }

    $total = array_sum($prices);

    // Если скидка больше общей стоимости товара, то возвращаем пустой массив
    if ($discount >= $total) {
        return [];
    }

    foreach ($prices as &$price) {
        // Находим долю стоимости товара в заказе
        $shareOfCostPercent = $price * 100 / $total;
        // Распределение скидки по долям
        $shareOfCost = $discount * $shareOfCostPercent / 100;
        // Вычитаем долю скидки из стоимости товара
        $price = $price - $shareOfCost;
    }

    return $prices;
}

$productPrices = [1001, 7500, 4500, 820, 970, 551, 111, 329, 1113, 15_000];

$sumBefore = array_sum($productPrices);

$result = distribute_discount(1500, $productPrices);

$sumAfter = array_sum($result);

var_dump($sumBefore, $sumAfter);
```
Результат
```
dima@eab5e8a5339f:/var/www/laravel$ php discount_func.php 
int(31895)
float(30395)
```
Конечно возможны коллизии (общая стоимость товаров после применения скидок отличается от значения скидки) когда в стоимости товаров указаны еще и копейки. 
Можно подумать над дополнительными проверками (для выявления подобных случаев), и разницу добавлять к последнему товару.
Но в целом функция должна работает.
