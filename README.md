import requests
from flask import Flask, request, render_template_string

app = Flask(__name__)

API_KEY = "YOUR_API_KEY"  # Get one free from https://openweathermap.org/api

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Weather Dashboard</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { background: linear-gradient(120deg, #89f7fe, #66a6ff); min-height: 100vh; color: #fff; }
        .container { max-width: 700px; margin-top: 50px; background: rgba(0,0,0,0.3); padding: 30px; border-radius: 15px; }
        .temp { font-size: 3rem; font-weight: bold; }
        canvas { background: #fff; border-radius: 10px; }
    </style>
</head>
<body>
    <div class="container text-center">
        <h2 class="mb-4">🌤️ Weather Dashboard</h2>
        <form method="post" class="d-flex mb-4">
            <input name="city" class="form-control me-2" placeholder="Enter city name" required>
            <button class="btn btn-light">Search</button>
        </form>

        {% if weather %}
            <div class="mb-4">
                <h3>{{ weather.city }}, {{ weather.country }}</h3>
                <div class="temp">{{ weather.temp }}°C</div>
                <p>{{ weather.desc }}</p>
                <img src="https://openweathermap.org/img/wn/{{ weather.icon }}@2x.png">
            </div>

            <h5 class="mb-3">Next 5 Days Forecast</h5>
            <canvas id="forecastChart" width="600" height="300"></canvas>

            <script>
                const ctx = document.getElementById('forecastChart');
                new Chart(ctx, {
                    type: 'line',
                    data: {
                        labels: {{ forecast.days | safe }},
                        datasets: [{
                            label: 'Temperature (°C)',
                            data: {{ forecast.temps | safe }},
                            borderColor: '#007bff',
                            borderWidth: 2,
                            fill: true,
                            backgroundColor: 'rgba(0,123,255,0.2)',
                            tension: 0.3
                        }]
                    },
                    options: { scales: { y: { beginAtZero: false } } }
                });
            </script>
        {% endif %}
    </div>
</body>
</html>
"""

def get_weather(city):
    try:
        current_url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric"
        forecast_url = f"https://api.openweathermap.org/data/2.5/forecast?q={city}&appid={API_KEY}&units=metric"

        current_data = requests.get(current_url).json()
        forecast_data = requests.get(forecast_url).json()

        weather = {
            "city": current_data["name"],
            "country": current_data["sys"]["country"],
            "temp": round(current_data["main"]["temp"]),
            "desc": current_data["weather"][0]["description"].title(),
            "icon": current_data["weather"][0]["icon"]
        }

        days, temps = [], []
        for item in forecast_data["list"][::8]:
            days.append(item["dt_txt"].split(" ")[0])
            temps.append(round(item["main"]["temp"]))

        forecast = {"days": days, "temps": temps}
        return weather, forecast
    except:
        return None, None

@app.route("/", methods=["GET", "POST"])
def home():
    weather, forecast = None, None
    if request.method == "POST":
        city = request.form.get("city")
        weather, forecast = get_weather(city)
    return render_template_string(HTML_TEMPLATE, weather=weather, forecast=forecast)

if __name__ == "__main__":
    app.run(debug=True)
