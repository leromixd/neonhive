<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Dice Box RPG</title>
    <style>
        :root {
            --gold: #ffcc00;
            --bg: #1a1a1a;
            --panel: #2a2a2a;
            --red: #e74c3c;
        }

        body {
            background: var(--bg);
            color: white;
            font-family: 'Segoe UI', sans-serif;
            display: flex;
            justify-content: center;
            padding: 20px;
            margin: 0;
        }

        .main-layout {
            display: flex;
            gap: 20px;
            max-width: 1100px;
            width: 100%;
        }

        /* Панель настройки */
        .settings {
            width: 260px;
            background: var(--panel);
            padding: 20px;
            border-radius: 15px;
            border: 1px solid #444;
        }

        .input-group { margin-bottom: 15px; }
        label { display: block; font-size: 12px; color: #888; margin-bottom: 5px; }
        input, select {
            width: 100%; padding: 10px; background: #1a1a1a; border: 1px solid #444;
            color: white; border-radius: 6px; box-sizing: border-box;
        }

        /* Центр: Коробка с кубами */
        .box-area {
            flex: 1;
            background: #111;
            padding: 25px;
            border-radius: 20px;
            border: 2px dashed #444;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .dice-collection {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            justify-content: center;
            width: 100%;
            min-height: 200px;
            margin-bottom: 30px;
        }

        /* Вид отдельного кубика в коробке */
        .die-item {
            position: relative;
            width: 80px;
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #222;
            border-radius: 10px;
            border: 1px solid #333;
        }

        .die-item svg { position: absolute; width: 100%; height: 100%; fill: #34495e; stroke: var(--gold); stroke-width: 2; }
        .die-item .num { position: relative; z-index: 5; font-weight: bold; font-size: 20px; color: var(--gold); }
        
        .remove-die {
            position: absolute; top: -8px; right: -8px;
            background: var(--red); color: white; border: none;
            width: 20px; height: 20px; border-radius: 50%;
            cursor: pointer; font-size: 12px; line-height: 1;
        }

        /* Кнопка БРОСОК */
        .roll-all-btn {
            width: 100%;
            padding: 20px;
            font-size: 24px;
            font-weight: bold;
            background: var(--gold);
            color: black;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            transition: 0.3s;
            box-shadow: 0 0 20px rgba(255, 204, 0, 0.3);
        }

        .roll-all-btn:hover { transform: scale(1.02); background: #ffdb4d; }
        .roll-all-btn:disabled { background: #555; cursor: not-allowed; box-shadow: none; }

        /* История */
        .history {
            width: 240px;
            background: var(--panel);
            padding: 20px;
            border-radius: 15px;
            border: 1px solid #444;
            display: flex;
            flex-direction: column;
        }

        .history-log {
            flex: 1; overflow-y: auto; height: 350px;
            list-style: none; padding: 0; margin: 10px 0;
            font-family: monospace; border-bottom: 1px solid #333;
        }

        .log-item { display: flex; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #222; font-size: 13px; }
        .sum-box { text-align: center; font-size: 22px; font-weight: bold; color: var(--gold); margin-top: 10px; }

        /* Анимация тряски */
        @keyframes shake {
            0% { transform: translate(2px, 2px); }
            50% { transform: translate(-2px, -2px); }
            100% { transform: translate(0, 0); }
        }
        .rolling { animation: shake 0.1s infinite; }
        .hide { display: none; }
    </style>
</head>
<body>

<div class="main-layout">
    <!-- ЛЕВО: Добавление -->
    <div class="settings">
        <h3 style="color:var(--gold); margin-top:0;">Все кубы IT TOP</h3>
        <div class="input-group">
            <label>Граней (d2 - d100)</label>
            <input type="number" id="sides-in" value="6">
        </div>
        <div class="input-group">
            <label>Визуальная форма</label>
            <select id="shape-in">
                <option value="4">Треугольник (d4)</option>
                <option value="6" selected>Квадрат (d6)</option>
                <option value="8">Алмаз (d8)</option>
                <option value="10">Ромб (d10)</option>
                <option value="20">Гексагон (d20)</option>
                <option value="100">Круг (d100)</option>
            </select>
        </div>
        <button onclick="addDie()" style="width:100%; background:#27ae60; color:white; border:none; padding:10px; border-radius:5px; cursor:pointer; font-weight:bold;">ДОБАВИТЬ В КОРОБКУ</button>
    </div>

    <!-- ЦЕНТР: Коробка -->
    <div class="box-area">
        <h2 style="margin-top:0;">Кубы</h2>
        <div id="dice-collection" class="dice-collection">
            <!-- Здесь будут кубики -->
            <p id="empty-msg" style="color:#555; margin-top:50px;">Кубов нет. Добавьте кубики слева.</p>
        </div>
        <button id="roll-btn" class="roll-all-btn" onclick="rollAll()" disabled>БРОСИТЬ ВСЕ</button>
    </div>

    <!-- ПРАВО: История -->
    <div class="history">
        <strong>ИСТОРИЯ БРОСКОВ</strong>
        <ul id="history-log" class="history-log"></ul>
        <div class="sum-box">СУММА: <span id="total-sum">0</span></div>
        <button onclick="clearAll()" style="margin-top:15px; background:none; border:1px solid #555; color:#888; cursor:pointer; padding:5px;">Очистить всё</button>
    </div>
</div>

<script>
    const paths = {
        4: 'M 50,5 L 95,90 L 5,90 Z',
        6: 'M 15,1 5 H 85 V 85 H 15 Z',
        8: 'M 50,5 L 90,50 L 50,95 L 10,50 Z M 10,50 L 90,50 M 50,5 L 50,95',
        10: 'M 50,5 L 90,45 L 50,95 L 10,45 Z',
        20: 'M 50,5 L 90,25 L 90,75 L 50,95 L 10,75 L 10,25 Z',
        100: 'M 50,50 m -45,0 a 45,45 0 1,0 90,0 a 45,45 0 1,0 -90,0'
    };

    let diceBox = []; // Список кубиков в коробке
    let totalScore = 0;
    let isRolling = false;

    const collectionEl = document.getElementById('dice-collection');
    const rollBtn = document.getElementById('roll-btn');
    const historyLog = document.getElementById('history-log');
    const sumEl = document.getElementById('total-sum');
    const emptyMsg = document.getElementById('empty-msg');

    // Функция добавления в коробку
    function addDie() {
        const sides = parseInt(document.getElementById('sides-in').value);
        const shape = document.getElementById('shape-in').value;
        
        if (sides < 2) return alert("Минимум 2 грани!");

        diceBox.push({ sides, shape, lastResult: '?' });
        updateUI();
    }

    // Удаление из коробки
    function removeDie(index) {
        if (isRolling) return;
        diceBox.splice(index, 1);
        updateUI();
    }

    // Обновление вида коробки
    function updateUI() {
        collectionEl.innerHTML = '';
        if (diceBox.length === 0) {
            collectionEl.appendChild(emptyMsg);
            rollBtn.disabled = true;
        } else {
            rollBtn.disabled = false;
            diceBox.forEach((die, index) => {
                const div = document.createElement('div');
                div.className = 'die-item';
                div.id = `die-${index}`;
                div.innerHTML = `
                    <button class="remove-die" onclick="removeDie(${index})">×</button>
                    <svg viewBox="0 0 100 100"><path d="${paths[die.shape]}" /></svg>
                    <div class="num" id="val-${index}">${die.lastResult}</div>
                `;
                collectionEl.appendChild(div);
            });
        }
    }

    // БРОСОК ВСЕХ КУБОВ
    function rollAll() {
        if (isRolling || diceBox.length === 0) return;
        isRolling = true;
        rollBtn.disabled = true;

        // Запускаем анимацию для всех
        diceBox.forEach((_, i) => {
            document.getElementById(`die-${i}`).classList.add('rolling');
        });

        let ticks = 0;
        let rollTimer = setInterval(() => {
            diceBox.forEach((die, i) => {
                document.getElementById(`val-${i}`).innerText = Math.floor(Math.random() * die.sides) + 1;
            });
            ticks++;

            if (ticks > 15) {
                clearInterval(rollTimer);
                finalizeRoll();
            }
        }, 60);
    }

    function finalizeRoll() {
        let currentRollSum = 0;
        let rollDetails = [];

        diceBox.forEach((die, i) => {
            const res = Math.floor(Math.random() * die.sides) + 1;
            die.lastResult = res;
            document.getElementById(`val-${i}`).innerText = res;
            document.getElementById(`die-${i}`).classList.remove('rolling');
            
            currentRollSum += res;
            rollDetails.push(`d${die.sides}:${res}`);
        });

        // Добавляем в историю
        totalScore += currentRollSum;
        sumEl.innerText = totalScore;

        const li = document.createElement('li');
        li.className = 'log-item';
        li.innerHTML = `<span title="${rollDetails.join(', ')}">Бросок (${diceBox.length} куб.)</span> <span>+${currentRollSum}</span>`;
        historyLog.prepend(li);

        isRolling = false;
        rollBtn.disabled = false;
    }

    function clearAll() {
        diceBox = [];
        totalScore = 0;
        sumEl.innerText = "0";
        historyLog.innerHTML = "";
        updateUI();
    }

    // Инициализация
    updateUI();
</script>

</body>
</html>
