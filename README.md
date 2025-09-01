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
            
            /* Background Image Properties */
            background-image: url('pakistan_bg.jpg');
            background-size: cover;
            background-position: center;
            background-attachment: fixed; /* Parallax effect ke liye */
            color: #fff; /* Text color ko white kiya hai background ke hisab se */
        }
        .header-container {
            text-align: center;
            margin-bottom: 20px;
            background-color: rgba(46, 125, 50, 0.7); /* Thoda transparent dark green */
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
            background: rgba(255, 255, 255, 0.85); /* Thoda transparent white */
            padding: 20px 30px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-bottom: 10px;
        }
        .counter-container button {
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
        }
        .chart-container {
            width: 80%;
            max-width: 700px;
            background: rgba(255, 255, 255, 0.85); /* Thoda transparent white */
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-top: 30px;
        }

        /* Mobile-friendly adjustments */
        @media (max-width: 768px) {
            .header-container h1 {
                font-size: 2em;
            }
            .header-container p {
                font-size: 1em;
            }
            .counter-container {
                flex-direction: column;
                gap: 10px;
                padding: 15px 20px;
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
        let counts = JSON.parse(localStorage.getItem('dailyCounts')) || {
            date: new Date().toLocaleDateString(),
            button1: 0,
            button2: 0,
            button3: 0,
            button4: 0
        };

        const today = new Date().toLocaleDateString();
        if (counts.date !== today) {
            counts = {
                date: today,
                button1: 0,
                button2: 0,
                button3: 0,
                button4: 0
            };
        }

        const button1 = document.getElementById('button1');
        const counter1 = document.getElementById('counter1');
        const button2 = document.getElementById('button2');
        const counter2 = document.getElementById('counter2');
        const button3 = document.getElementById('button3');
        const counter3 = document.getElementById('counter3');
        const button4 = document.getElementById('button4');
        const counter4 = document.getElementById('counter4');

        counter1.textContent = counts.button1;
        counter2.textContent = counts.button2;
        counter3.textContent = counts.button3;
        counter4.textContent = counts.button4;

        const ctx = document.getElementById('dailyChart').getContext('2d');
        const dailyChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['Alhamdulillah', 'Jazakallah', 'Mashallah', 'Inshallah'],
                datasets: [{
                    label: 'Daily Clicks',
                    data: [counts.button1, counts.button2, counts.button3, counts.button4],
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
            localStorage.setItem('dailyCounts', JSON.stringify(counts));
            dailyChart.data.datasets[0].data = [counts.button1, counts.button2, counts.button3, counts.button4];
            dailyChart.update();
        }

        button1.addEventListener('click', () => {
            counts.button1++;
            counter1.textContent = counts.button1;
            updateData();
        });

        button2.addEventListener('click', () => {
            counts.button2++;
            counter2.textContent = counts.button2;
            updateData();
        });
        
        button3.addEventListener('click', () => {
            counts.button3++;
            counter3.textContent = counts.button3;
            updateData();
        });

        button4.addEventListener('click', () => {
            counts.button4++;
            counter4.textContent = counts.button4;
            updateData();
        });
    </script>
</body>

