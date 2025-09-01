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
            width: 300px;
            justify-content: space-between;
        }
        .counter-container button {
            flex-grow: 1;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            background-color: #4CAF50;
            color: white;
            transition: background-color 0.3s;
        }
        .counter-container button:hover {
            background-color: #388E3C;
        }
        .counter {
            font-size: 24px;
            font-weight: bold;
            color: #2E7D32;
            min-width: 40px;
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
        .button-group-container {
            display: flex;
            gap: 10px;
            margin-top: 20px;
        }
        .button-group-container button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            color: white;
            transition: background-color 0.3s;
        }
        #saveButton {
            background-color: #2196F3;
        }
        #saveButton:hover {
            background-color: #1976D2;
        }
        #resetButton {
            background-color: #F44336;
        }
        #resetButton:hover {
            background-color: #D32F2F;
        }
        .chart-data-container {
            margin-top: 20px;
            font-size: 1em;
            color: #2E7D32;
            width: 80%;
            max-width: 700px;
            background: rgba(255, 255, 255, 0.85);
            padding: 20px;
            border-radius: 10px;
        }
        .date-data {
            font-weight: bold;
            margin-bottom: 5px;
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
            .button-group-container {
                flex-direction: column;
                gap: 10px;
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
    
    <div class="counter-container">
        <button id="button5">Ya Rahamkallah</button>
        <div class="counter" id="counter5">0</div>
    </div>
    
    <div class="counter-container">
        <button id="button6">Subhanallah</button>
        <div class="counter" id="counter6">0</div>
    </div>

    <div class="button-group-container">
        <button id="saveButton">Save ✔️</button>
        <button id="resetButton">Reset 🔄</button>
    </div>

    <div class="chart-container">
        <canvas id="dailyChart"></canvas>
    </div>

    <div class="chart-data-container" id="data-display"></div>

    <script>
        const buttonNames = ['Alhamdulillah', 'Jazakallah', 'Mashallah', 'Inshallah', 'Ya Rahamkallah', 'Subhanallah'];
        const buttonIds = ['button1', 'button2', 'button3', 'button4', 'button5', 'button6'];
        const counterIds = ['counter1', 'counter2', 'counter3', 'counter4', 'counter5', 'counter6'];

        let historicalData = JSON.parse(localStorage.getItem('historicalCounts')) || {};
        let today = new Date().toLocaleDateString();

        if (!historicalData[today]) {
            historicalData[today] = {};
            buttonNames.forEach(name => historicalData[today][name] = 0);
        }

        buttonNames.forEach((name, index) => {
            document.getElementById(counterIds[index]).textContent = historicalData[today][name];
        });

        const ctx = document.getElementById('dailyChart').getContext('2d');
        const dailyChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: Object.keys(historicalData).sort((a, b) => new Date(a) - new Date(b)),
                datasets: buttonNames.map((name, index) => ({
                    label: name,
                    data: Object.keys(historicalData).sort((a, b) => new Date(a) - new Date(b)).map(date => historicalData[date][name]),
                    borderColor: ['#4CAF50', '#388E3C', '#2E7D32', '#1B5E20', '#A5D6A7', '#66BB6A'][index],
                    backgroundColor: ['rgba(76, 175, 80, 0.2)', 'rgba(56, 142, 60, 0.2)', 'rgba(46, 125, 50, 0.2)', 'rgba(27, 94, 32, 0.2)', 'rgba(165, 214, 167, 0.2)', 'rgba(102, 187, 106, 0.2)'][index],
                    borderWidth: 2,
                    fill: false,
                    tension: 0.1
                }))
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    title: {
                        display: true,
                        text: 'Performance Over Time'
                    },
                    tooltip: {
                        callbacks: {
                            title: (tooltipItem) => {
                                return `Date: ${tooltipItem[0].label}`;
                            },
                            label: (tooltipItem) => {
                                const datasetLabel = tooltipItem.dataset.label;
                                const value = tooltipItem.raw;
                                return `${datasetLabel}: ${value}`;
                            }
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        title: {
                            display: true,
                            text: 'Clicks'
                        }
                    },
                    x: {
                        title: {
                            display: true,
                            text: 'Date'
                        }
                    }
                }
            }
        });

        function updateData() {
            localStorage.setItem('historicalCounts', JSON.stringify(historicalData));
            
            const updatedLabels = Object.keys(historicalData).sort((a, b) => new Date(a) - new Date(b));
            dailyChart.data.labels = updatedLabels;
            
            buttonNames.forEach((name, index) => {
                dailyChart.data.datasets[index].data = updatedLabels.map(date => historicalData[date][name]);
            });

            dailyChart.update();
            displayTableData();
        }

        buttonIds.forEach((id, index) => {
            document.getElementById(id).addEventListener('click', () => {
                const buttonName = buttonNames[index];
                if (typeof historicalData[today][buttonName] === 'undefined') {
                     historicalData[today][buttonName] = 0;
                }
                historicalData[today][buttonName]++;
                document.getElementById(counterIds[index]).textContent = historicalData[today][buttonName];
                updateData();
            });
        });

        const saveButton = document.getElementById('saveButton');
        saveButton.addEventListener('click', () => {
            localStorage.setItem('historicalCounts', JSON.stringify(historicalData));
            alert("Data saved successfully!");
        });
        
        const resetButton = document.getElementById('resetButton');
        resetButton.addEventListener('click', () => {
            if (confirm("Kya aap sach mein aaj ke counters ko reset karna chahte hain?")) {
                buttonNames.forEach(name => historicalData[today][name] = 0);
                
                buttonNames.forEach((name, index) => {
                    document.getElementById(counterIds[index]).textContent = 0;
                });
                updateData();
                alert("Data reset for today.");
            }
        });
        
        function displayTableData() {
            const dataDisplay = document.getElementById('data-display');
            dataDisplay.innerHTML = '';
            
            const chartLabels = Object.keys(historicalData).sort((a, b) => new Date(a) - new Date(b));

            chartLabels.forEach(date => {
                const dateDiv = document.createElement('div');
                dateDiv.classList.add('date-data');
                dateDiv.innerHTML = `**Date: ${date}**`;
                dataDisplay.appendChild(dateDiv);
                
                const ul = document.createElement('ul');
                buttonNames.forEach(name => {
                    const li = document.createElement('li');
                    li.textContent = `${name}: ${historicalData[date][name]}`;
                    ul.appendChild(li);
                });
                dataDisplay.appendChild(ul);
            });
        }
        
        updateData();
    </script>
</body>
</html>
