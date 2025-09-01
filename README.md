<!DOCTYPE html>
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
            height: 100vh;
            background-color: #e8f5e9;
            margin: 0;
            gap: 20px;
        }
        .header-container {
            text-align: center;
            margin-bottom: 20px;
        }
        .header-container h1 {
            color: #2e7d32;
            font-size: 3em;
            margin: 0;
        }
        .header-container p {
            color: #4caf50;
            font-size: 1.2em;
            margin-top: 5px;
        }
        .counter-container {
            display: flex;
            align-items: center;
            gap: 15px;
            background: #fff;
            padding: 20px 30px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
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
            background: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            margin-top: 30px;
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
        <button id="button1">Button 1</button>
        <div class="counter" id="counter1">0</div>
    </div>

    <div class="counter-container">
        <button id="button2">Button 2</button>
        <div class="counter" id="counter2">0</div>
    </div>

    <div class="chart-container">
        <canvas id="dailyChart"></canvas>
    </div>

    <script>
        // Initialize counts from local storage or set to 0
        let counts = JSON.parse(localStorage.getItem('dailyCounts')) || {
            date: new Date().toLocaleDateString(),
            button1: 0,
            button2: 0
        };

        // Check if the date has changed
        const today = new Date().toLocaleDateString();
        if (counts.date !== today) {
            counts = {
                date: today,
                button1: 0,
                button2: 0
            };
        }

        // DOM Elements
        const button1 = document.getElementById('button1');
        const counter1 = document.getElementById('counter1');
        const button2 = document.getElementById('button2');
        const counter2 = document.getElementById('counter2');

        // Update UI with initial counts
        counter1.textContent = counts.button1;
        counter2.textContent = counts.button2;

        // Chart setup
        const ctx = document.getElementById('dailyChart').getContext('2d');
        const dailyChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['Button 1', 'Button 2'],
                datasets: [{
                    label: 'Daily Clicks',
                    data: [counts.button1, counts.button2],
                    backgroundColor: [
                        'rgba(76, 175, 80, 0.6)',
                        'rgba(56, 142, 60, 0.6)'
                    ],
                    borderColor: [
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

        // Function to update local storage and chart
        function updateData() {
            localStorage.setItem('dailyCounts', JSON.stringify(counts));
            dailyChart.data.datasets[0].data = [counts.button1, counts.button2];
            dailyChart.update();
        }

        // Event listeners for buttons
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
    </script>
</body>
</html></html>
