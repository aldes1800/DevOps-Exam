Microservice Experiment — Flask Status Service

Doel

In dit experiment maak ik een eenvoudige microservice met Flask.

De microservice heeft één taak:

De status van de service teruggeven.

app.py

from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/status")
def status():
    return jsonify({
        "service": "user-service",
        "status": "running"
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

Uitvoeren

Installeer Flask:

pip3 install flask

Start de microservice:

python3 app.py

Testen

Gebruik curl:

curl http://localhost:5000/status

Verwachte output:

{
  "service": "user-service",
  "status": "running"
}

Hoe werkt het?

Client
↓
HTTP GET
↓
/status
↓
Flask Microservice
↓
JSON response

Wat demonstreer ik?

een kleine zelfstandige service;

één duidelijke verantwoordelijkheid;

een REST-endpoint;

JSON-output;

testen met curl.

Mondelinge uitleg

Ik heb een eenvoudige Flask-microservice gemaakt met één specifieke verantwoordelijkheid. De service biedt een /status endpoint aan en geeft via JSON terug dat de service actief is.