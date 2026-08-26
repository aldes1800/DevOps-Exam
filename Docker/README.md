Docker Lab — Build a Sample Web App in a Docker Container

Doel van de lab

In deze lab heb ik een eenvoudige webapp gebouwd met Python en Flask en deze daarna verpakt in een Docker-container.

De lab bestaat uit zes delen:

DEVASC VM starten.

Een eenvoudig Bash-script maken.

Een eenvoudige Flask-webapp maken.

HTML en CSS toevoegen aan de webapp.

Een Bash-script maken dat automatisch een Dockerfile aanmaakt, een Docker image bouwt en een container start.

De Docker-container bouwen, uitvoeren, testen, onderzoeken, stoppen, herstarten en verwijderen.

Gebruikte technologieën:

Bash

Python 3

Flask

HTML/CSS

Docker

Dockerfile

Docker networking

curl

docker exec

docker logs

1. Omgeving

Werkmap:

~/labs/devnet-src/sample-app

Belangrijkste bestanden:

sample-app/
├── sample_app.py
├── sample-app.sh
├── user-input
├── static/
│   └── style.css
└── templates/
    └── index.html

2. Eerste Bash-script

Bestand:

user-input.sh

Inhoud:

#!/bin/bash

echo -n "Enter Your Name: "
read userName
echo "Your name is $userName."

Test:

bash user-input.sh

Voorbeeld:

Enter Your Name: Alan
Your name is Alan.

Daarna werd het script executable gemaakt:

chmod a+x user-input.sh

en hernoemd:

mv user-input.sh user-input

Daarna kon het rechtstreeks worden gestart:

./user-input

3. Eerste Flask-webapp

Flask werd geïnstalleerd met:

pip3 install flask

Eerste versie van sample_app.py:

from flask import Flask
from flask import request

sample = Flask(__name__)

@sample.route("/")
def main():
    return "You are calling me from " + request.remote_addr + "\n"

if __name__ == "__main__":
    sample.run(host="0.0.0.0", port=8080)

Starten:

python3 sample_app.py

Test:

curl http://0.0.0.0:8080

Resultaat:

You are calling me from 127.0.0.1

4. HTML en CSS toevoegen

templates/index.html:

<html>
<head>
  <title>Sample app</title>
  <link rel="stylesheet" href="/static/style.css" />
</head>
<body>
  <h1>You are calling me from {{request.remote_addr}}</h1>
</body>
</html>

static/style.css:

body {background: lightsteelblue;}

Daarna werd sample_app.py aangepast:

from flask import Flask
from flask import request
from flask import render_template

sample = Flask(__name__)

@sample.route("/")
def main():
    return render_template("index.html")

if __name__ == "__main__":
    sample.run(host="0.0.0.0", port=8080)

De webpagina toonde nu dezelfde tekst als een H1 op een lichtblauwe achtergrond.

5. Bash-script voor Docker

sample-app.sh maakt eerst een tijdelijke buildmap:

#!/bin/bash

mkdir tempdir
mkdir tempdir/templates
mkdir tempdir/static

cp sample_app.py tempdir/.
cp -r templates/* tempdir/templates/.
cp -r static/* tempdir/static/.

Daarna wordt automatisch een Dockerfile gegenereerd.

Werkende versie in mijn VM:

echo "FROM python:3.11-slim" > tempdir/Dockerfile
echo "RUN pip install --no-cache-dir --progress-bar off flask" >> tempdir/Dockerfile
echo "COPY ./static /home/myapp/static/" >> tempdir/Dockerfile
echo "COPY ./templates /home/myapp/templates/" >> tempdir/Dockerfile
echo "COPY sample_app.py /home/myapp/" >> tempdir/Dockerfile
echo "EXPOSE 8080" >> tempdir/Dockerfile
echo "CMD python3 /home/myapp/sample_app.py" >> tempdir/Dockerfile

Daarna:

cd tempdir
docker build -t sampleapp .
docker run -t -d -p 8080:8080 --name samplerunning sampleapp
docker ps -a

6. Docker image bouwen

De image werd gebouwd met:

docker build -t sampleapp .

Succesvolle output:

Successfully built ...
Successfully tagged sampleapp:latest

7. Docker container starten

De container werd gestart met:

docker run -t -d -p 8080:8080 --name samplerunning sampleapp

Betekenis:

-t maakt terminaltoegang mogelijk;

-d draait de container op de achtergrond;

-p 8080:8080 koppelt hostpoort 8080 aan containerpoort 8080;

--name samplerunning geeft de container een vaste naam;

sampleapp is de Docker image.

Controle:

docker ps

Resultaat bevatte:

sampleapp
Up ...
0.0.0.0:8080->8080/tcp
samplerunning

8. Webapp vanuit Docker testen

Test:

curl http://localhost:8080

Response:

<h1>You are calling me from 172.17.0.1</h1>

De webapp draaide nu in Docker. Daarom werd niet langer 127.0.0.1, maar het Docker bridge-adres gebruikt.

9. Docker bridge netwerk

Met:

ip address

werd de interface:

docker0

gevonden met:

inet 172.17.0.1/16

Dit is de standaard Docker bridge-interface op de host.

10. Container onderzoeken

De draaiende container werd geopend met:

docker exec -it samplerunning /bin/bash

De applicatiebestanden werden gecontroleerd:

ls /home/myapp/

Resultaat:

sample_app.py
static
templates

Daarna werd de container verlaten met:

exit

11. Container stoppen, starten en verwijderen

Stoppen:

docker stop samplerunning

Controle:

docker ps -a

Daarna opnieuw starten:

docker start samplerunning

Controleren:

docker ps

Definitief verwijderen:

docker stop samplerunning
docker rm samplerunning
docker ps -a

Na docker rm stond samplerunning niet meer in de lijst.

Problemen en oplossingen

RuntimeError: can't start new thread

Tijdens de Docker-build en later bij Flask requests kreeg ik:

RuntimeError: can't start new thread

De DEVASC VM had moeite met extra threads in de moderne Python/pip omgeving.

Oplossing voor de build:

FROM python:3.11-slim
RUN pip install --no-cache-dir --progress-bar off flask

Oplossing voor Flask:

sample.run(host="0.0.0.0", port=8080, threaded=False)

Hierdoor kon Flask requests verwerken zonder voor iedere request een nieuwe thread te starten.

Dockerfile-instructies stonden op één regel

Tijdens een mislukte build verscheen:

RUN pip install ... flask COPY ./static ...

waardoor pip meldde:

ERROR: Invalid requirement: './static'

De oplossing was elke Dockerfile-instructie op een afzonderlijke regel te genereren.

Unable to find image 'sampleapp:latest'

Deze fout verscheen nadat een eerdere Docker-build al was mislukt.

Omdat de image niet succesvol gebouwd was, kon:

docker run ... sampleapp

de image niet vinden.

Na een succesvolle build verdween deze fout.

curl: (52) Empty reply from server

De container stond wel Up, maar de webapp antwoordde niet.

Met:

docker logs samplerunning

werd de echte oorzaak gevonden:

RuntimeError: can't start new thread

Daarna werd Flask aangepast met:

threaded=False

en werkte de webapp correct.

Belangrijke Docker-commando's

docker build -t sampleapp .
docker run -t -d -p 8080:8080 --name samplerunning sampleapp
docker ps
docker ps -a
docker logs samplerunning
docker exec -it samplerunning /bin/bash
docker stop samplerunning
docker start samplerunning
docker rm samplerunning

Belangrijke begrippen voor het examen

Docker image

Een Docker image is een template waaruit containers gemaakt worden.

In deze lab:

sampleapp

Docker container

Een container is een draaiende instantie van een Docker image.

In deze lab:

samplerunning

Dockerfile

Een Dockerfile bevat de instructies om een Docker image te bouwen.

Belangrijke instructies:

FROM
RUN
COPY
EXPOSE
CMD

FROM

Bepaalt de basisimage.

RUN

Voert een commando uit tijdens de build.

COPY

Kopieert bestanden naar de image.

EXPOSE

Documenteert welke poort de applicatie gebruikt.

CMD

Bepaalt welk commando standaard wordt gestart wanneer de container draait.

Port mapping

-p 8080:8080

betekent:

host 8080 → container 8080

docker exec

Opent of voert een commando uit in een draaiende container.

docker logs

Toont de logs van een container en is belangrijk voor troubleshooting.

Wat heb ik geleerd?

Na deze lab begrijp ik:

hoe Bash-scripts werken;

wat een shebang is;

hoe executable permissions werken;

hoe Flask een eenvoudige webapp kan aanbieden;

hoe Flask templates rendert;

hoe HTML en CSS aan Flask gekoppeld worden;

wat een Dockerfile is;

hoe een Docker image gebouwd wordt;

hoe een Docker container gestart wordt;

hoe port mapping werkt;

hoe Docker bridge networking werkt;

hoe ik een container intern kan onderzoeken;

hoe ik Docker logs gebruik voor troubleshooting;

hoe ik containers stop, start en verwijder;

hoe Bash een Docker-buildproces kan automatiseren.

Aanbevolen GitHub-structuur

Docker/
└── Do1-Sample-Web-App/
    ├── README.md
    ├── sample_app.py
    ├── sample-app.sh
    ├── user-input
    ├── static/
    │   └── style.css
    ├── templates/
    │   └── index.html
    └── screenshots/
        ├── 01-bash-script-working.png
        ├── 02-flask-curl-working.png
        ├── 03-flask-html-css-working.png
        ├── 04-docker-build-success.png
        ├── 05-docker-container-running.png
        ├── 06-docker-webapp-curl-success.png
        ├── 07-docker0-network.png
        ├── 08-docker-exec-container.png
        └── 09-container-stop-start-remove.png

Mondeling examen — korte uitleg

In deze lab heb ik eerst met Bash en Flask een eenvoudige webapp gemaakt die het IP-adres van de client toont. Daarna heb ik HTML en CSS toegevoegd. Vervolgens heb ik met een Bash-script automatisch een Dockerfile gegenereerd, een Docker image gebouwd en een container gestart. De container publiceerde poort 8080 naar de host. Daarna heb ik de app getest, het Docker bridge-netwerk bekeken, de container intern onderzocht met docker exec en de container gestopt, opnieuw gestart en verwijderd. Tijdens de lab heb ik ook een resourceprobleem opgelost met docker logs en de Flask-server aangepast zodat hij zonder threading kon werken.

Conclusie

De volledige workflow was:

Bash
↓
Flask
↓
HTML/CSS
↓
Dockerfile
↓
Docker Image
↓
Docker Container
↓
Port Mapping
↓
Docker Networking
↓
Testing & Troubleshooting

Het eindresultaat was een werkende Flask-webapp in een Docker-container die via poort 8080 bereikbaar was.