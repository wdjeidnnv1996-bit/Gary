<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Кнопка Заказа Еды</title>
    
    <style>
        /* --- CSS: Стиль кнопки --- */
        
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f4f4f4;
            flex-direction: column;
        }

        #orderButton {
            /* Базовый стиль */
            background-color: #ff5722; /* Яркий оранжевый */
            color: white;
            border: none;
            padding: 15px 30px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 18px;
            font-weight: bold;
            margin: 40px 2px;
            cursor: pointer;
            border-radius: 8px; /* Скругленные углы */
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            transition: background-color 0.3s ease, transform 0.1s ease;
        }

        #orderButton:hover {
            /* Эффект при наведении */
            background-color: #e64a19; /* Чуть темнее */
        }

        #orderButton:active {
            /* Эффект при клике */
            transform: scale(0.98);
        }

        #statusMessage {
            margin-top: 20px;
            font-size: 1.1em;
            color: #333;
            min-height: 30px; /* Чтобы избежать "прыжков" */
        }
    </style>
</head>
<body>

    <button id="orderButton">
        🛒 Заказать еду сейчас!
    </button>
    
    <div id="statusMessage">Нажмите, чтобы оформить заказ.</div>

    <script>
        // --- JAVASCRIPT: Логика кнопки ---
        
        // Получаем ссылку на кнопку и элемент сообщения
        const orderButton = document.getElementById('orderButton');
        const statusMessage = document.getElementById('statusMessage');

        // Имитация данных о заказе
        const currentOrder = {
            item: "Пицца 'Маргарита'",
            quantity: 1,
            total: 550, // Стоимость в рублях/гривнах/тнг
            deliveryTime: randomTime(25, 45) // Случайное время доставки
        };

        // Вспомогательная функция для генерации случайного времени
        function randomTime(min, max) {
            return Math.floor(Math.random() * (max -
