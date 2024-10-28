<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Документация: Система Заказов и Билетов</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            line-height: 1.6;
            color: #333;
        }
        h1 {
            color: #0056b3;
        }
        h2 {
            color: #333;
        }
        .section {
            margin-bottom: 30px;
        }
        .section h3 {
            color: #0056b3;
        }
        code {
            color: #d6336c;
            background-color: #f9f9f9;
            padding: 2px 4px;
            border-radius: 3px;
        }
        .description {
            background-color: #f9f9f9;
            padding: 10px;
            border-left: 5px solid #0056b3;
            margin-top: -10px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 15px 0;
            font-size: 1rem;
            border: 1px solid #ddd;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border: 1px solid #ddd;
        }
        th {
            background-color: #f4f4f4;
            font-weight: bold;
        }
    </style>
</head>
<body>

<h1>Документация по системе заказов и билетов</h1>

<div class="section">
    <h2>Функция добавления заказа</h2>
    <p>Функция <code>addOrder</code> используется для добавления нового заказа в базу данных с проверкой внешних API на бронирование и подтверждение заказа.</p>
</div>

<div class="section">
    <h3>Основные шаги функции</h3>
    <ul>
        <li><strong>Генерация баркода:</strong> Для каждого заказа генерируется уникальный <code>barcode</code> длиной 13 символов с помощью функции <code>generateBarcode</code>.</li>
        <li><strong>Отправка бронирования:</strong> Запрос отправляется на <code>https://api.site.com/book</code> с основными данными заказа. Ответ проверяется на успешное бронирование или ошибку (если баркод уже существует).</li>
        <li><strong>Повторная генерация:</strong> При ошибке повторно генерируется баркод до трех раз, чтобы избежать бесконечного повторения при технической ошики со строны api.</li>
        <li><strong>Подтверждение заказа:</strong> Если бронирование успешно, выполняется запрос подтверждения на <code>https://api.site.com/approve</code>.</li>
        <li><strong>Сохранение в базу данных:</strong> Если заказ подтвержден, он сохраняется в таблицу <code>orders_list</code>.</li>
    </ul>
</div>

<div class="section">
    <h3>Пример вызова функции</h3>
    <pre>
$response = addOrder(1, '2024-11-01', 50.0, 2, 25.0, 1);
echo json_encode($response);
    </pre>
</div>

<div class="section">
    <h2>Таблицы базы данных</h2>

    <h3>Таблица <code>orders</code> (заказы)</h3>
    <table>
        <thead>
            <tr>
                <th>Поле</th>
                <th>Тип</th>
                <th>Описание</th>
            </tr>
        </thead>
        <tbody>
            <tr><td>id</td><td>INT</td><td>Уникальный ID заказа</td></tr>
            <tr><td>event_id</td><td>INT</td><td>ID события</td></tr>
            <tr><td>event_date</td><td>DATETIME</td><td>Дата и время события</td></tr>
            <tr><td>user_id</td><td>INT</td><td>ID пользователя</td></tr>
            <tr><td>equal_price</td><td>INT</td><td>Общая сумма заказа</td></tr>
            <tr><td>created</td><td>DATETIME</td><td>Дата создания заказа</td></tr>
        </tbody>
    </table>

    <h3>Таблица <code>ticket_types</code> (типы билетов)</h3>
    <table>
        <thead>
            <tr>
                <th>Поле</th>
                <th>Тип</th>
                <th>Описание</th>
            </tr>
        </thead>
        <tbody>
            <tr><td>id</td><td>INT</td><td>Уникальный ID типа билета</td></tr>
            <tr><td>name</td><td>VARCHAR</td><td>Название типа билета (взрослый, детский, льготный, групповой)</td></tr>
            <tr><td>price</td><td>INT</td><td>Цена за единицу билета</td></tr>
        </tbody>
    </table>

    <h3>Таблица <code>tickets</code> (билеты)</h3>
    <table>
        <thead>
            <tr>
                <th>Поле</th>
                <th>Тип</th>
                <th>Описание</th>
            </tr>
        </thead>
        <tbody>
            <tr><td>id</td><td>INT</td><td>Уникальный ID билета</td></tr>
            <tr><td>order_id</td><td>INT</td><td>ID заказа</td></tr>
            <tr><td>ticket_type_id</td><td>INT</td><td>ID типа билета</td></tr>
            <tr><td>barcode</td><td>VARCHAR</td><td>Уникальный баркод билета</td></tr>
        </tbody>
    </table>
</div>

<div class="section">
    <h2>Обоснование нормализации</h2>
    <p>Таблицы <code>orders</code>, <code>ticket_types</code>, и <code>tickets</code> позволяют хранить данные эффективно.</p>
    <ul>
        <li><strong>Таблица <code>ticket_types</code>:</strong> Включает все возможные типы билетов (например, взрослый, детский, льготный, групповой), что упрощает добавление новых типов.</li>
        <li><strong>Таблица <code>tickets</code>:</strong> Связана с <code>orders</code> и <code>ticket_types</code> и содержит индивидуальные баркоды для каждого билета. Это позволяет контролировать каждого посетителя отдельно.</li>
    </ul>
</div>

</body>
</html>
