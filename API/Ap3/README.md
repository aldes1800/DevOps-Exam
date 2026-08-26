Python API Experiment — Weather API

Doel

In dit experiment haal ik met Python actuele weergegevens voor Brussel op via de Open-Meteo REST API.

Ik demonstreer hiermee:

een HTTP GET request;

query parameters;

een JSON-response;

statuscodecontrole;

data uit JSON uitlezen.

Python-script

Bestand:

weather_api.py

Code:

import requests

url = "https://api.open-meteo.com/v1/forecast"

params = {
    "latitude": 50.8503,
    "longitude": 4.3517,
    "current": "temperature_2m,wind_speed_10m"
}

response = requests.get(url, params=params)

if response.status_code == 200:
    data = response.json()

    temperature = data["current"]["temperature_2m"]
    wind = data["current"]["wind_speed_10m"]

    print("Weather in Brussels")
    print("-------------------")
    print(f"Temperature: {temperature} °C")
    print(f"Wind speed: {wind} km/h")
else:
    print("API request failed:", response.status_code)

Uitvoeren

python3 weather_api.py

Voorbeeldoutput:

Weather in Brussels
-------------------
Temperature: 18.4 °C
Wind speed: 12.1 km/h

De waarden veranderen afhankelijk van het actuele weer.

Hoe werkt het?

De flow is:

Python
↓
GET request
↓
Open-Meteo REST API
↓
JSON response
↓
Temperatuur en windsnelheid uitlezen

Belangrijke regels:

requests.get(url, params=params)

stuurt het API-request.

response.status_code == 200

controleert of de request succesvol was.

response.json()

zet de JSON-response om naar Python-data.

Mondelinge uitleg

Ik heb een Python-script gemaakt dat via een GET-request de Open-Meteo REST API aanspreekt. Ik stuur latitude en longitude als parameters mee, controleer de HTTP-statuscode en verwerk daarna de JSON-response. Vervolgens toon ik de actuele temperatuur en windsnelheid van Brussel.

Bestanden

Python-API-Experiment/
├── README.md
└── weather_api.py