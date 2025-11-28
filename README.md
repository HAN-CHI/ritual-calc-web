<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>禮儀日期速查工具</title>
    
    <script src="https://cdn.jsdelivr.net/npm/dayjs@1/dayjs.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/lunar-javascript@1.1.2/dist/lunar-javascript.min.js"></script>

    <style>
        /* -------------------------- */
        /* 手機優化 CSS (與前次相同) */
        /* -------------------------- */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f4f4f9;
        }
        .container {
            max-width: 600px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        label, input, button {
            display: block;
            width: 100%;
            margin-bottom: 15px;
            font-size: 1.1em;
        }
        input[type="date"] {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            background-color: #007bff;
            color: white;
            padding: 12px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover {
            background-color: #0056b3;
        }
        #results {
            margin-top: 30px;
            padding-top: 15px;
            border-top: 1px solid #eee;
        }
        #results p {
            font-size: 1.1em;
            line-height: 1.6;
        }
        #results span {
            font-weight: bold;
            color: #cc0000;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>亡者日期計算</h1>
        
        <label for="death-date">亡故日期 (國曆)：</label>
        <input type="date" id="death-date">

        <label for="birth-date">出生日期 (國曆)：</label>
        <input type="date" id="birth-date">
        
        <button id="calculate-btn">開始計算</button>

        <div id="results">
            <h2>計算結果</h2>
            <p><strong>亡者農曆生日：</strong> <span id="result-lunar-birth">--</span></p>
            <hr>
            <p><strong>百日 (第100天)：</strong> <span id="result-hundred">--</span></p>
            <p><strong>國曆對年 (滿一年)：</strong> <span id="result-anniversary">--</span></p>
            <p><strong>農曆對年 (次年同農曆日)：</strong> <span id="result-lunar-anniversary">--</span></p>
        </div>
    </div>
    
    <script>
        // --------------------------
        // 核心 JavaScript 邏輯
        // --------------------------
        
        // 注意：這裡使用 window.dayjs 和 window.Lunar/window.Solar，因為是透過 <script> 標籤載入的
        const dayjs = window.dayjs;
        const { Solar, Lunar } = window.lunar;

        // 取得 DOM 元素
        const deathDateInput = document.getElementById('death-date');
        const birthDateInput = document.getElementById('birth-date');
        const calculateBtn = document.getElementById('calculate-btn');

        // 取得結果顯示元素
        const resultLunarBirth = document.getElementById('result-lunar-birth');
        const resultHundred = document.getElementById('result-hundred');
        const resultAnniversary = document.getElementById('result-anniversary');
        const resultLunarAnniversary = document.getElementById('result-lunar-anniversary');

        function calculateRitualDates() {
            const deathDateYMD = deathDateInput.value;
            const birthDateYMD = birthDateInput.value;

            if (!deathDateYMD || !birthDateYMD) {
                alert("請輸入完整的亡故日期和出生日期！");
                return;
            }

            const [dYear, dMonth, dDay] = deathDateYMD.split('-').map(Number);
            const [bYear, bMonth, bDay] = birthDateYMD.split('-').map(Number);

            // --- 1. 計算亡者農曆生日 (轉換) ---
            try {
                const solarBirth = Solar.fromYmd(bYear, bMonth, bDay);
                const lunarBirth = solarBirth.getLunar();
                resultLunarBirth.textContent = 
                    `${lunarBirth.getYearInChinese()}年 ${lunarBirth.getMonthInChinese()}月 ${lunarBirth.getDayInChinese()}日 (${lunarBirth.getYearZodiac()}年)`;
            } catch (e) {
                resultLunarBirth.textContent = '轉換錯誤';
            }

            // --- 2. 計算百日 (加 99 天) ---
            const hundredthDay = dayjs(deathDateYMD).add(99, 'day');
            resultHundred.textContent = hundredthDay.format('YYYY年M月D日');

            // --- 3. 計算國曆對年 (加 1 年) ---
            const firstAnniversary = dayjs(deathDateYMD).add(1, 'year');
            resultAnniversary.textContent = firstAnniversary.format('YYYY年M月D日');

            // --- 4. 計算農曆對年 (次年同一農曆日) ---
            try {
                const solarDeath = Solar.fromYmd(dYear, dMonth, dDay);
                const lunarDeath = solarDeath.getLunar();
                
                const lunarY = lunarDeath.getYear();
                const lunarM = lunarDeath.getMonth();
                const lunarD = lunarDeath.getDay();

                const nextYearLunar = Lunar.fromYmd(lunarY + 1, lunarM, lunarD);
                const lunarAnniversarySolar = nextYearLunar.getSolar();
                
                resultLunarAnniversary.textContent = 
                    `${lunarAnniversarySolar.toYmd()} (國曆) / ${nextYearLunar.getYearInChinese()}年 ${nextYearLunar.getMonthInChinese()}月 ${nextYearLunar.getDayInChinese()}日 (農曆)`;
            } catch (e) {
                resultLunarAnniversary.textContent = '計算錯誤 (請檢查日期是否在有效範圍)';
            }
        }

        // 綁定按鈕事件
        calculateBtn.addEventListener('click', calculateRitualDates);
    </script>
</body>
</html>
