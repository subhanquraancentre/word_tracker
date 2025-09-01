<WORD TRACKER>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Subhan Quran - Counter</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            gap: 20px;
            
            background-image: url('quran_bg.jpg');
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
            color: #fff;
        }
        .header-container {
            text-align: center;
            margin-bottom: 20px;
            background-color: rgba(46, 125, 50, 0.7);
            padding: 20px;
            border-radius: 10px;
        }
        .header-container h1 {
            color: #e8f5e9;
            font-size: 3em;
            margin: 0;
        }
        .header-container p {
            color: #c8e6c9;
            font-size: 1.2em;
            margin-top: 5px;
        }
        .counter-container {
            display: flex;
            align-items: center;
            gap: 15px;
            background: rgba(255, 255, 255, 0.85);
            padding: 20px 30px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-bottom: 10px;
            width: 300px; /* Buttons ka size ek jaisa rakhne ke liye */
            justify-content: space-between;
        }
        .counter-container button {
            flex-grow: 1; /* Button ko poori jagah dene ke liye */
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            background-color: #4caf50;
            color: white;
            transition: background-color 0.3s;
        }
        .counter-container button:hover {
            background-color: #388e3c;
        }
        .counter {
            font-size: 24px;
            font-weight: bold;
            color: #2e7d32;
            min-width: 40px; /* Counter ke liye fixed space */
            text-align: right;
        }
        .chart-container {
            width: 80%;
            max-width: 700px;
            background: rgba(255, 255, 255, 0.85);
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-top: 30px;
        }

        @media (max-width: 768px) {
            .header-container h1 {
                font-size: 2em;
            }
            .header-container p {
                font-size: 1em;
            }
            .counter-container {
                width: 90%;
                flex-direction: row;
                justify-content: space-between;
                padding: 15px 20px;
            }
            .counter-container button {
                flex-grow: 0;
                width: 65%;
            }
            .chart-container {
                width: 95%;
            }
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <div class="header-container">
        <h1>Subhan Quran</h1>
        <p>Mustafa Ali Bukhari</p>
    </div>

    <div class="counter-container">
        <button id="button1">Alhamdulillah</button>
        <div class="counter" id="counter1">0</div>
    </div>

    <div class="counter-container">
        <button id="button2">Jazakallah</button>
        <div class="counter" id="counter2">0</div>
    </div>

    <div class="counter-container">
        <button id="button3">Mashallah</button>
        <div class="counter" id="counter3">0</div>
    </div>

    <div class="counter-container">
        <button id="button4">Inshallah</button>
        <div class="counter" id="counter4">0</div>
    </div>

    <div class="chart-container">
        <canvas id="dailyChart"></canvas>
    </div>

    <script>
        const buttonNames = ['Alhamdulillah', 'Jazakallah', 'Mashallah', 'Inshallah'];
        const buttonIds = ['button1', 'button2', 'button3', 'button4'];
        const counterIds = ['counter1', 'counter2', 'counter3', 'counter4'];
        
        let historicalData = JSON.parse(localStorage.getItem('historicalCounts')) || {};
        let today = new Date().toLocaleDateString();
        
        if (!historicalData[today]) {
            historicalData[today] = {
                'Alhamdulillah': 0,
                'Jazakallah': 0,
                'Mashallah': 0,
                'Inshallah': 0
            };
        }

        buttonNames.forEach((name, index) => {
            document.getElementById(counterIds[index]).textContent = historicalData[today][name];
        });

        const ctx = document.getElementById('dailyChart').getContext('2d');
        const dailyChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: buttonNames,
                datasets: [{
                    label: today + ' Clicks',
                    data: buttonNames.map(name => historicalData[today][name]),
                    backgroundColor: [
                        'rgba(76, 175, 80, 0.6)',
                        'rgba(56, 142, 60, 0.6)',
                        'rgba(76, 175, 80, 0.6)',
                        'rgba(56, 142, 60, 0.6)'
                    ],
                    borderColor: [
                        'rgba(76, 175, 80, 1)',
                        'rgba(56, 142, 60, 1)',
                        'rgba(76, 175, 80, 1)',
                        'rgba(56, 142, 60, 1)'
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true,
                        title: {
                            display: true,
                            text: 'Clicks'
                        }
                    }
                },
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    title: {
                        display: true,
                        text: 'Daily Click Chart'
                    }
                }
            }
        });

        function updateData() {
            localStorage.setItem('historicalCounts', JSON.stringify(historicalData));
            dailyChart.data.datasets[0].data = buttonNames.map(name => historicalData[today][name]);
            dailyChart.update();
        }

        buttonIds.forEach((id, index) => {
            document.getElementById(id).addEventListener('click', () => {
                const buttonName = buttonNames[index];
                historicalData[today][buttonName]++;
                document.getElementById(counterIds[index]).textContent = historicalData[today][buttonName];
                updateData();
            });
        });
    </script>
</body>
