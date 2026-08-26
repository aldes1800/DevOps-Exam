Python API Experiment 2 — Weather Webform

Doel

In dit experiment combineer ik Python, Flask, een webformulier en een REST API.

De gebruiker vult een stad in. De applicatie zoekt de locatie op en toont daarna de actuele temperatuur.

Bestanden

Weather-Webform/
├── app.py
├── README.md
└── templates/
    └── index.html

app.py

from flask import Flask, render_template, request
import requests

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    result = None

    if request.method == "POST":
        city = request.form["city"]

        geo = requests.get(
            "https://geocoding-api.open-meteo.com/v1/search",
            params={"name": city, "count": 1}
        ).json()

        if geo.get("results"):
            location = geo["results"][0]

            weather = requests.get(
                "https://api.open-meteo.com/v1/forecast",
                params={
                    "latitude": location["latitude"],
                    "longitude": location["longitude"],
                    "current": "temperature_2m"
                }
            ).json()

            result = (
                f'{location["name"]}: '
                f'{weather["current"]["temperature_2m"]} °C'
            )
        else:
            result = "City not found."

    return render_template("index.html", result=result)

if __name__ == "__main__":
    app.run(debug=True)

templates/index.html

<!DOCTYPE html>
<html>
<head>
    <title>Weather API</title>
</head>
<body>

<h1>Check the Weather</h1>

<form method="POST">
    <input type="text" name="city" placeholder="Enter a city" required>
    <button type="submit">Search</button>
</form>

{% if result %}
    <h2>{{ result }}</h2>
{% endif %}

</body>
</html>

Uitvoeren

Installeer de nodige packages:

pip3 install flask requests

Start de applicatie:

python3 app.py

Open daarna:

http://127.0.0.1:5000

Hoe werkt het?

Webformulier
↓
POST request
↓
Flask
↓
Geocoding API
↓
Weather API
↓
JSON response
↓
Resultaat op webpagina

Wat heb ik geleerd?

formulierdata uitlezen met Flask;

GET en POST gebruiken;

een REST API aanspreken vanuit Python;

JSON verwerken;

API-resultaten tonen in HTML.

Mondelinge uitleg

Ik heb een Flask-webformulier gemaakt waarin de gebruiker een stad invoert. Python leest de formulierdata uit, zoekt via een REST API de coördinaten van de stad op en haalt daarna de actuele temperatuur op. Het resultaat wordt opnieuw op de webpagina weergegeven.