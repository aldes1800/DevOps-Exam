Docker Image Experiment — Flask Webservice

Doel

In dit experiment maak ik een eenvoudige Flask-webservice, verpak ik die in een Docker image en start ik er een container van.

app.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from my Docker webservice!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, threaded=False)

Dockerfile

FROM python:3.11-slim

WORKDIR /app

COPY app.py .

RUN pip install --no-cache-dir --progress-bar off flask

EXPOSE 5000

CMD ["python3", "app.py"]

Image bouwen

docker build -t mywebservice .

Verwachte output:

Successfully built ...
Successfully tagged mywebservice:latest

Container starten

docker run -d -p 5000:5000 --name webservice mywebservice

Testen

curl http://localhost:5000

Verwachte output:

Hello from my Docker webservice!

Problemen en oplossingen

1. RuntimeError: can't start new thread

Tijdens:

RUN pip install flask

kreeg ik:

RuntimeError: can't start new thread

Oplossing:

RUN pip install --no-cache-dir --progress-bar off flask

2. curl: (52) Empty reply from server

De container draaide, maar Flask kon geen nieuwe thread starten voor de request.

Oplossing in app.py:

app.run(host="0.0.0.0", port=5000, threaded=False)

Daarna heb ik opnieuw gebouwd:

docker stop webservice
docker rm webservice
docker build -t mywebservice .
docker run -d -p 5000:5000 --name webservice mywebservice

Hoe werkt het?

Flask webservice
↓
Dockerfile
↓
Docker image
↓
Container
↓
Webservice op poort 5000

Mondelinge uitleg

Ik heb een Flask-webservice gemaakt en deze in een Docker image verpakt. Daarna heb ik een container gestart op poort 5000 en de webservice getest met curl. Tijdens het experiment heb ik ook een thread-probleem opgelost door de pip progress bar uit te schakelen en Flask zonder threading te starten.