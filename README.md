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
            text-align: center;
            min-width: 120px;
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
            width: 90%;
            max-width: 900px;
            height: 400px;
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
            padding: 8px 16px;
            font-size: 14px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            color: white;
            transition: background-color 0.3s;
        }
        #saveButton {
            background-color: #4CAF50;
        }
        #saveButton:hover {
            background-color: #388E3C;
        }
        #resetButton {
            background-color: #4CAF50;
        }
        #resetButton:hover {
            background-color: #388E3C;
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
                height: 300px;
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

        let historicalData = JSON.parse(localStorage.getItem('historicalWeeklyCounts')) || {};
        
        const getTodayDay = () => {
            const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
            return days[new Date().getDay()];
        };

        const todayDay = getTodayDay();
        
        // Ensure today's day data structure is complete
        if (!historicalData[todayDay]) {
            historicalData[todayDay] = {};
            buttonNames.forEach(name => historicalData[todayDay][name] = 0);
        } else {
            buttonNames.forEach(name => {
                if (typeof historicalData[todayDay][name] === 'undefined') {
                    historicalData[todayDay][name] = 0;
                }
            });
        }
        
        // Initialize counters from stored data
        buttonNames.forEach((name, index) => {
            document.getElementById(counterIds[index]).textContent = historicalData[todayDay][name];
        });
        
        const chartColors = [
            '#4CAF50',
            '#66BB6A',
            '#81C784',
            '#A5D6A7',
            '#C8E6C9',
            '#E8F5E9'
        ];

        const ctx = document.getElementById('dailyChart').getContext('2d');
        const dailyChart = new Chart(ctx, {
            type: 'bar', // Ab line chart ke bajaye bar chart hai
            data: {
                labels: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'],
                datasets: buttonNames.map((name, index) => ({
                    label: name,
                    data: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'].map(day => historicalData[day]?.[name] || 0),
                    borderColor: chartColors[index],
                    backgroundColor: chartColors[index] + '40',
                    borderWidth: 2,
                }))
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    title: {
                        display: true,
                        text: 'Weekly Performance',
                        color: '#2E7D32',
                        font: {
                            size: 20,
                            weight: 'bold'
                        }
                    },
                    tooltip: {
                        callbacks: {
                            title: (tooltipItem) => {
                                return `Day: ${tooltipItem[0].label}`;
                            },
                            label: (tooltipItem) => {
                                const datasetLabel = dailyChart.data.datasets[tooltipItem.datasetIndex].label;
                                const value = tooltipItem.raw;
                                return `${datasetLabel}: ${value}`;
                            }
                        },
                        backgroundColor: 'rgba(46, 125, 50, 0.9)',
                        bodyColor: '#fff',
                        titleColor: '#fff'
                    },
                    legend: {
                        display: true,
                        position: 'bottom',
                        labels: {
                            color: '#2E7D32',
                            font: {
                                size: 14
                            },
                            boxWidth: 20
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        title: {
                            display: true,
                            text: 'Counts',
                            color: '#2E7D32',
                            font: {
                                size: 16
                            }
                        },
                        ticks: {
                            color: '#2E7D32',
                            font: {
                                size: 12
                            }
                        },
                        grid: {
                            color: 'rgba(224, 224, 224, 0.5)'
                        }
                    },
                    x: {
                        title: {
                            display: true,
                            text: 'Day of the Week',
                            color: '#2E7D32',
                            font: {
                                size: 16
                            }
                        },
                        ticks: {
                            color: '#2E7D32',
                            font: {
                                size: 12
                            }
                        },
                        grid: {
                            color: 'rgba(224, 224, 224, 0.5)'
                        }
                    }
                }
            }
        });

        function updateData() {
            localStorage.setItem('historicalWeeklyCounts', JSON.stringify(historicalData));
            
            const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
            
            dailyChart.data.labels = daysOfWeek;
            
            buttonNames.forEach((name, index) => {
                dailyChart.data.datasets[index].data = daysOfWeek.map(day => historicalData[day]?.[name] || 0);
            });

            dailyChart.update();
            displayTableData();
        }

        buttonIds.forEach((id, index) => {
            document.getElementById(id).addEventListener('click', () => {
                const buttonName = buttonNames[index];
                historicalData[todayDay][buttonName]++;
                document.getElementById(counterIds[index]).textContent = historicalData[todayDay][buttonName];
                updateData();
            });
        });

        const saveButton = document.getElementById('saveButton');
        saveButton.addEventListener('click', () => {
            localStorage.setItem('historicalWeeklyCounts', JSON.stringify(historicalData));
            alert("Data saved successfully!");
        });
        
        const resetButton = document.getElementById('resetButton');
        resetButton.addEventListener('click', () => {
            if (confirm("Kya aap sach mein aaj ke counters ko reset karna chahte hain?")) {
                buttonNames.forEach(name => historicalData[todayDay][name] = 0);
                
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
            
            const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

            daysOfWeek.forEach(day => {
                const dayData = historicalData[day];
                if (dayData) {
                    const dayDiv = document.createElement('div');
                    dayDiv.classList.add('date-data');
                    dayDiv.innerHTML = `**Day: ${day}**`;
                    dataDisplay.appendChild(dayDiv);
                    
                    const ul = document.createElement('ul');
                    buttonNames.forEach(name => {
                        if (dayData[name] !== undefined) {
                            const li = document.createElement('li');
                            li.textContent = `${name}: ${dayData[name]}`;
                            ul.appendChild(li);
                        }
                    });
                    dataDisplay.appendChild(ul);
                }
            });
        }
        
        updateData();
    </script>
</body>
</html>
