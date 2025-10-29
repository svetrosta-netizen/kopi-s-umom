# kopi-s-umom<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Копи с умом: рубль или евро?</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .calculator {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h2 {
            color: #2c3e50;
            text-align: center;
            margin-bottom: 30px;
        }
        .inputs {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-bottom: 30px;
        }
        .input-group {
            display: flex;
            flex-direction: column;
        }
        label {
            margin-bottom: 5px;
            font-weight: bold;
            color: #34495e;
        }
        input {
            padding: 10px;
            border: 1px solid #bdc3c7;
            border-radius: 5px;
            font-size: 16px;
        }
        button {
            background: #3498db;
            color: white;
            border: none;
            padding: 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            transition: background 0.3s;
            grid-column: 1 / -1;
        }
        button:hover {
            background: #2980b9;
        }
        .results {
            margin-top: 30px;
            overflow-x: auto;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 30px;
        }
        th, td {
            padding: 12px;
            text-align: right;
            border-bottom: 1px solid #ecf0f1;
        }
        th {
            background: #34495e;
            color: white;
            position: sticky;
            top: 0;
        }
        tr:hover {
            background: #f8f9fa;
        }
        .positive {
            color: #27ae60;
            font-weight: bold;
        }
        .negative {
            color: #e74c3c;
            font-weight: bold;
        }
        .chart-container {
            height: 400px;
            margin-top: 30px;
        }
        @media (max-width: 768px) {
            .inputs {
                grid-template-columns: 1fr;
            }
            th, td {
                padding: 8px 4px;
                font-size: 14px;
            }
        }
    </style>
</head>
<body>
    <div class="calculator">
        <h2>🧠 Копи с умом: рубль или евро?</h2>
        
        <div class="inputs">
            <div class="input-group">
                <label>Начальная сумма (₽):</label>
                <input type="number" id="initialAmount" value="500000" min="0">
            </div>
            
            <div class="input-group">
                <label>Ежемесячное пополнение (₽):</label>
                <input type="number" id="monthlyDeposit" value="10000" min="0">
            </div>
            
            <div class="input-group">
                <label>Срок (лет):</label>
                <input type="number" id="years" value="5" min="1" max="30">
            </div>
            
            <button onclick="calculate()">Рассчитать накопления</button>
        </div>

        <div class="results">
            <table id="resultsTable">
                <thead>
                    <tr>
                        <th>Год</th>
                        <th>Рубли (номинал)</th>
                        <th>Profit Max (евро)</th>
                        <th>Рубли (реальная стоимость)</th>
                        <th>Цена бездействия</th>
                    </tr>
                </thead>
                <tbody id="tableBody">
                    <!-- Таблица заполнится после расчета -->
                </tbody>
            </table>
        </div>

        <div class="chart-container">
            <canvas id="growthChart"></canvas>
        </div>
    </div>

    <script>
        let chart = null;

        function calculate() {
            const initialAmount = parseFloat(document.getElementById('initialAmount').value);
            const monthlyDeposit = parseFloat(document.getElementById('monthlyDeposit').value);
            const years = parseInt(document.getElementById('years').value);

            if (isNaN(initialAmount) || isNaN(monthlyDeposit) || isNaN(years)) {
                alert('Пожалуйста, заполните все поля корректно');
                return;
            }

            const tableBody = document.getElementById('tableBody');
            tableBody.innerHTML = '';

            const yearlyData = [];
            const labels = [];
            const rubleNominalData = [];
            const profitMaxData = [];
            const rubleRealData = [];
            const inactionCostData = [];

            let rubleNominal = initialAmount;
            let profitMax = initialAmount;
            let rubleReal = initialAmount;

            const monthlyRateRub = 0.10 / 12;
            const monthlyRateEur = 0.22 / 12;
            const months = years * 12;

            // Расчет с ежемесячной капитализацией
            for (let year = 1; year <= years; year++) {
                // Расчет с ежемесячными пополнениями
                for (let month = 1; month <= 12; month++) {
                    rubleNominal = rubleNominal * (1 + monthlyRateRub) + monthlyDeposit;
                    profitMax = profitMax * (1 + monthlyRateEur) + monthlyDeposit;
                }

                // Реальная стоимость рублевых накоплений
                rubleReal = rubleNominal / Math.pow(1.12, year);
                const inactionCost = profitMax - rubleReal;

                const rowData = {
                    year: year,
                    rubleNominal: rubleNominal,
                    profitMax: profitMax,
                    rubleReal: rubleReal,
                    inactionCost: inactionCost
                };

                yearlyData.push(rowData);
                labels.push(`Год ${year}`);
                rubleNominalData.push(rubleNominal);
                profitMaxData.push(profitMax);
                rubleRealData.push(rubleReal);
                inactionCostData.push(inactionCost);

                // Добавляем строку в таблицу
                const row = document.createElement('tr');
                
                row.innerHTML = `
                    <td>${year}</td>
                    <td>${formatCurrency(rubleNominal)}</td>
                    <td class="positive">${formatCurrency(profitMax)}</td>
                    <td class="negative">${formatCurrency(rubleReal)}</td>
                    <td class="positive">${formatCurrency(inactionCost)}</td>
                `;
                
                tableBody.appendChild(row);
            }

            updateChart(labels, rubleNominalData, profitMaxData, rubleRealData);
        }

        function formatCurrency(amount) {
            return new Intl.NumberFormat('ru-RU', { 
                style: 'currency', 
                currency: 'RUB',
                maximumFractionDigits: 0
            }).format(amount);
        }

        function updateChart(labels, rubleNominal, profitMax, rubleReal) {
            const ctx = document.getElementById('growthChart').getContext('2d');
            
            if (chart) {
                chart.destroy();
            }

            chart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            label: 'Рубли (номинал)',
                            data: rubleNominal,
                            borderColor: '#95a5a6',
                            backgroundColor: 'rgba(149, 165, 166, 0.1)',
                            borderWidth: 2,
                            tension: 0.1
                        },
                        {
                            label: 'Profit Max (евро)',
                            data: profitMax,
                            borderColor: '#27ae60',
                            backgroundColor: 'rgba(39, 174, 96, 0.1)',
                            borderWidth: 3,
                            tension: 0.1
                        },
                        {
                            label: 'Рубли (реальная стоимость)',
                            data: rubleReal,
                            borderColor: '#e74c3c',
                            backgroundColor: 'rgba(231, 76, 60, 0.1)',
                            borderWidth: 2,
                            tension: 0.1
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        title: {
                            display: true,
                            text: 'Динамика накоплений по годам'
                        },
                        tooltip: {
                            mode: 'index',
                            intersect: false,
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    label += formatCurrency(context.parsed.y);
                                    return label;
                                }
                            }
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: function(value) {
                                    return formatCurrency(value);
                                }
                            }
                        }
                    }
                }
            });
        }

        // Первоначальный расчет при загрузке
        document.addEventListener('DOMContentLoaded', calculate);
    </script>
</body>
</html>
