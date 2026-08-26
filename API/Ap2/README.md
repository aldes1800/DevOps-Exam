Ap2 — Integrate a REST API in a Python Application

Doel van de lab

In deze lab heb ik een Python-applicatie gebouwd die gebruikmaakt van de GraphHopper REST API.

De applicatie kan:

een vertrekpunt opvragen;

een bestemming opvragen;

locaties omzetten naar latitude en longitude;

een route tussen twee locaties berekenen;

afstand en reistijd weergeven;

stap-voor-stap route-instructies tonen;

verschillende vervoersmiddelen gebruiken;

fouten van de API verwerken.

De applicatie gebruikt twee GraphHopper API's:

Geocoding API

Routing API

De uiteindelijke applicatie ondersteunt de voertuigprofielen:

car

bike

foot

Het programma is tijdens de lab stap voor stap opgebouwd in verschillende Python-bestanden.

1. Gebruikte technologieën

Voor deze lab heb ik gebruikt:

Python 3

Visual Studio Code

DEVASC VM

GraphHopper REST API

JSON

HTTP GET requests

Python requests

Python urllib.parse

2. Eerste Python-script

Het eerste bestand was:

graphhopper_parse-json_1.py

Hierin werden de nodige Python-modules geïmporteerd:

import requests
import urllib.parse

requests wordt gebruikt om HTTP-requests naar de API te sturen.

urllib.parse wordt gebruikt om parameters correct om te zetten naar een geldige URL.

3. GraphHopper Geocoding API

De eerste API waarmee ik werkte was de Geocoding API.

Een locatie zoals:

Washington, D.C.

wordt door de API omgezet naar geografische coördinaten:

latitude
longitude

Een API-request wordt bijvoorbeeld opgebouwd met:

url = geocode_url + urllib.parse.urlencode({
    "q": location,
    "limit": "1",
    "key": key
})

Daarna wordt het HTTP GET-request uitgevoerd:

replydata = requests.get(url)

Het JSON-resultaat wordt verwerkt met:

json_data = replydata.json()

4. JSON verwerken

GraphHopper stuurt de resultaten terug als JSON.

Uit deze data worden bijvoorbeeld latitude en longitude gehaald:

lat = json_data["hits"][0]["point"]["lat"]
lng = json_data["hits"][0]["point"]["lng"]

Daarnaast worden ook locatiegegevens opgehaald:

name = json_data["hits"][0]["name"]
value = json_data["hits"][0]["osm_value"]

Indien beschikbaar worden ook state en country gebruikt om een duidelijkere locatienaam te maken.

Voorbeeld:

Bruxelles - Brussel,
Région de Bruxelles-Capitale,
Belgique / Belgique / Belgien

5. Geocoding-functie

De geocoding-code werd in een functie geplaatst:

def geocoding(location, key):

Hierdoor kan dezelfde code worden gebruikt voor zowel:

Starting Location

als:

Destination

De functie retourneert:

statuscode
latitude
longitude
locatienaam

Voorbeeld:

(200, 50.8467372, 4.352493, "Brussels ...")

HTTP-statuscode 200 betekent dat de API-request succesvol werd uitgevoerd.

6. User input

In plaats van locaties hardcoded in de code te plaatsen, vraagt de applicatie de gebruiker om invoer:

loc1 = input("Starting Location: ")
loc2 = input("Destination: ")

Een while True-loop zorgt ervoor dat meerdere routes achter elkaar kunnen worden opgevraagd.

De applicatie kan worden afgesloten met:

q

of:

quit

Hiervoor wordt break gebruikt:

if loc1 == "quit" or loc1 == "q":
    break

7. Lege invoer verwerken

Wanneer een gebruiker geen locatie invoert, vraagt het programma opnieuw om een locatie:

while location == "":
    location = input("Enter the location again: ")

Hierdoor wordt geen lege API-request verstuurd.

8. Geocoding error handling

De applicatie controleert zowel de HTTP-statuscode als het aantal resultaten.

if json_status == 200 and len(json_data["hits"]) != 0:

Dit is belangrijk omdat een API-call statuscode 200 kan teruggeven terwijl de lijst met gevonden locaties leeg is.

Bij een ongeldige API-key kan bijvoorbeeld het volgende terugkomen:

Geocode API status: 401
Error message: Wrong credentials

Hierdoor crasht het programma niet wanneer de API een fout terugstuurt.

9. GraphHopper Routing API

Nadat vertrekpunt en bestemming geocoded zijn, worden de coördinaten gebruikt om een route op te vragen.

Voor het vertrekpunt:

op = "&point=" + str(orig[1]) + "%2C" + str(orig[2])

Voor de bestemming:

dp = "&point=" + str(dest[1]) + "%2C" + str(dest[2])

Daarna wordt de Routing API URL opgebouwd en via een HTTP GET-request opgevraagd.

Bij een succesvolle route geeft GraphHopper:

Routing API Status: 200

10. Afstand

GraphHopper geeft de afstand terug in meter.

Deze waarde wordt omgezet naar kilometer:

km = paths_data["paths"][0]["distance"] / 1000

en miles:

miles = paths_data["paths"][0]["distance"] / 1000 / 1.61

Voorbeeld:

Distance Traveled: 34.1 miles / 55.0 km

11. Reistijd

GraphHopper geeft de reistijd terug in milliseconden.

Deze waarde wordt omgezet naar uren, minuten en seconden:

sec = int(paths_data["paths"][0]["time"] / 1000 % 60)

min = int(
    paths_data["paths"][0]["time"]
    / 1000
    / 60
    % 60
)

hr = int(
    paths_data["paths"][0]["time"]
    / 1000
    / 60
    / 60
)

Daarna wordt de tijd weergegeven als:

hh:mm:ss

Voorbeeld:

Trip Duration: 00:46:32

12. Route-instructies

GraphHopper geeft ook een lijst met route-instructies terug.

De applicatie doorloopt deze lijst met een for-loop:

for each in range(
    len(paths_data["paths"][0]["instructions"])
):

Per instructie worden twee gegevens opgehaald:

path = paths_data["paths"][0]["instructions"][each]["text"]

distance = paths_data["paths"][0]["instructions"][each]["distance"]

Voorbeeldoutput:

Continue onto ...
Turn left onto ...
Turn right onto ...
Keep left ...
Arrive at destination

Bij iedere instructie wordt ook de afstand in kilometer en miles weergegeven.

13. Routing API error handling

Niet tussen iedere twee locaties kan een route worden berekend.

Ik heb bijvoorbeeld getest met:

Starting Location: Beijing
Destination: Washington, D.C.

GraphHopper gaf:

Routing API Status: 400

en:

Error message: Connection between locations not found

De applicatie controleert daarom:

if paths_status == 200:

Wanneer de request mislukt, wordt de foutmelding uit de JSON-response weergegeven in plaats van dat het programma crasht.

14. Verschillende voertuigprofielen

In de laatste versie van de applicatie kan de gebruiker kiezen tussen:

car
bike
foot

De beschikbare profielen worden opgeslagen in:

profile = ["car", "bike", "foot"]

De gebruiker kiest een voertuig:

vehicle = input(
    "Enter a vehicle profile from the list above: "
)

De gekozen waarde wordt meegestuurd naar GraphHopper:

urllib.parse.urlencode({
    "key": key,
    "vehicle": vehicle
})

15. Ongeldige voertuigkeuze

Wanneer een gebruiker iets anders invult dan:

car
bike
foot

wordt automatisch car gebruikt.

else:
    vehicle = "car"
    print(
        "No valid vehicle profile was entered. "
        "Using the car profile."
    )

Dit werd getest met:

chair

De applicatie gaf:

No valid vehicle profile was entered. Using the car profile.

en berekende vervolgens correct een autoroute.

16. Test — car

Ik heb de applicatie getest met:

Vehicle: car
Starting Location: Brussels
Destination: Antwerp

Resultaat:

Routing API Status: 200
Distance Traveled: 34.1 miles / 55.0 km
Trip Duration: 00:46:32

Daarna werden de volledige route-instructies weergegeven.

17. Test — bike

Ik heb dezelfde route getest met:

Vehicle: bike
Starting Location: Brussels
Destination: Antwerp

Resultaat:

Routing API Status: 200
Distance Traveled: 31.3 miles / 50.4 km
Trip Duration: 02:54:17

Hieruit blijkt dat GraphHopper een andere route en reistijd berekent afhankelijk van het gekozen voertuigprofiel.

18. Test — foot

Daarna heb ik getest met:

Vehicle: foot
Starting Location: Brussels
Destination: Antwerp

Resultaat:

Routing API Status: 200
Distance Traveled: 28.2 miles / 45.4 km
Trip Duration: 09:07:29

Ook hier werden de bijbehorende wandelinstructies weergegeven.

19. Eindversie

De uiteindelijke versie van de applicatie is:

graphhopper_parse-json_7.py

De verschillende bestanden tonen de ontwikkeling van de applicatie:

graphhopper_parse-json_1.py
graphhopper_parse-json_2.py
graphhopper_parse-json_3.py
graphhopper_parse-json_4.py
graphhopper_parse-json_5.py
graphhopper_parse-json_6.py
graphhopper_parse-json_7.py

Problemen en oplossingen

SyntaxError — unmatched ')'

Tijdens het schrijven van een print()-statement had ik een haakje op de verkeerde plaats.

Fout:

print("Geocoding API URL for") + loc1 + ":\n" + url)

Correct:

print("Geocoding API URL for " + loc1 + ":\n" + url)

NameError — geocode_url

Tijdens het ombouwen van mijn code naar een functie kreeg ik:

NameError: name 'geocode_url' is not defined

Dit kwam doordat de variabele niet op de juiste plaats in de functie stond.

De oplossing was:

def geocoding(location, key):
    geocode_url = "https://graphhopper.com/api/1/geocode?"

NameError — paths_status

Tijdens het bouwen van de Routing API kreeg ik:

NameError: name 'paths_status' is not defined

De statuscode moest eerst worden opgeslagen:

paths_status = paths_reply.status_code

voordat deze gebruikt kon worden in:

if paths_status == 200:

Ongeldige locatie

Een willekeurige locatie zoals:

QSFSQDF

kan een succesvolle HTTP-request geven zonder zoekresultaten.

Daarom controleert de code:

json_status == 200 and len(json_data["hits"]) != 0

Hierdoor ontstaat geen:

IndexError: list index out of range

Geen route beschikbaar

Niet alle locaties kunnen via de Routing API verbonden worden.

Voor:

Beijing
→
Washington, D.C.

kreeg ik:

Routing API Status: 400
Error message: Connection between locations not found

De applicatie toont deze foutmelding nu op een gecontroleerde manier.

Wat heb ik geleerd?

Na deze lab begrijp ik:

wat een REST API is;

hoe Python een REST API kan aanspreken;

hoe requests.get() werkt;

hoe URL query parameters worden opgebouwd;

hoe urllib.parse.urlencode() werkt;

hoe JSON-response data in Python wordt verwerkt;

hoe nested JSON dictionaries en lists worden uitgelezen;

wat latitude en longitude zijn;

hoe een Geocoding API werkt;

hoe een Routing API werkt;

wat HTTP statuscodes zoals 200, 400 en 401 betekenen;

hoe API error handling werkt;

hoe if, else, while en for in een echte applicatie gebruikt worden;

hoe een Python-functie hergebruik mogelijk maakt;

hoe API-data kan worden geconverteerd en geformatteerd;

hoe verschillende API parameters verschillende resultaten produceren.

Security

De GraphHopper API-key wordt niet gepubliceerd in deze repository.

In de GitHub-versie van de code wordt daarom:

key = "YOUR_API_KEY"

gebruikt.

Een echte API-key hoort niet in een publieke GitHub repository of publieke screenshots te staan.

Aanbevolen GitHub-structuur

API/
└── Ap2-GraphHopper-REST-API/
    ├── README.md
    ├── graphhopper_parse-json_1.py
    ├── graphhopper_parse-json_2.py
    ├── graphhopper_parse-json_3.py
    ├── graphhopper_parse-json_4.py
    ├── graphhopper_parse-json_5.py
    ├── graphhopper_parse-json_6.py
    ├── graphhopper_parse-json_7.py
    └── screenshots/
        ├── 01-imports-working.png
        ├── 02-geocoding-response.png
        ├── 03-user-input.png
        ├── 04-geocoding-error-handling.png
        ├── 05-routing-success.png
        ├── 06-route-instructions.png
        ├── 07-routing-error.png
        └── 08-final-application.png

Screenshots

Voor deze lab zijn vooral de volgende screenshots nuttig als bewijs:

Python imports werken.

Een succesvolle Geocoding API-response.

Dynamische invoer van Starting Location en Destination.

Geocoding error handling.

Succesvolle Routing API-request met status 200.

Stap-voor-stap route-instructies.

Routing error met status 400.

Eindapplicatie met car, bike of foot.

Let op: upload geen screenshots waarop je echte GraphHopper API-key zichtbaar is.

Mondeling examen — korte samenvatting

Als ik deze lab kort moet uitleggen:

Ik heb in Python een applicatie gebouwd die de GraphHopper REST API aanspreekt. Eerst gebruik ik de Geocoding API om een vertrekpunt en bestemming om te zetten naar latitude en longitude. Daarna stuur ik die coördinaten naar de Routing API. De JSON-response verwerk ik in Python om de afstand, reistijd en stap-voor-stap route-instructies weer te geven. Ik heb daarnaast error handling en verschillende voertuigprofielen zoals car, bike en foot toegevoegd.

Belangrijke begrippen voor het examen

Wat is Geocoding?

Het omzetten van een tekstuele locatie zoals:

Brussels

naar geografische coördinaten:

latitude
longitude

Wat doet requests.get()?

requests.get(url)

Stuurt een HTTP GET-request naar de API.

Wat doet .json()?

replydata.json()

Zet de JSON-response om naar Python-objecten zodat de gegevens eenvoudig kunnen worden uitgelezen.

Wat betekent [0]?

Bij:

json_data["hits"][0]

is hits een lijst en [0] betekent dat het eerste resultaat wordt gebruikt.

HTTP 200

De request werd succesvol verwerkt.

HTTP 400

De request kon niet correct worden verwerkt, bijvoorbeeld omdat geen route beschikbaar is.

HTTP 401

Authenticatie is mislukt, bijvoorbeeld door een ongeldige API-key.

Waarom deze controle?

if json_status == 200 and len(json_data["hits"]) != 0:

Omdat een API-request technisch succesvol kan zijn terwijl er geen zoekresultaten aanwezig zijn.

Waarom een functie gebruiken?

def geocoding(location, key):

Dezelfde geocoding-logica is nodig voor zowel vertrekpunt als bestemming. Door een functie te gebruiken wordt dubbele code vermeden.

Wat doet urllib.parse.urlencode()?

Het zet parameters om naar een correcte querystring voor een URL.

Bijvoorbeeld:

Washington, D.C.

wordt gecodeerd zodat het veilig als URL-parameter kan worden verstuurd.

Wat doet de for-loop?

De for-loop doorloopt alle route-instructies uit:

paths_data["paths"][0]["instructions"]

en toont per stap de instructie en afstand.

Waarom vehicle?

De GraphHopper Routing API kan verschillende routes berekenen voor:

car
bike
foot

Het gekozen voertuig wordt als parameter meegestuurd naar de API.

Conclusie

In deze lab heb ik stap voor stap een Python-applicatie ontwikkeld die een externe REST API gebruikt.

Eerst heb ik de GraphHopper Geocoding API gebruikt om tekstuele locaties om te zetten naar latitude en longitude.

Daarna heb ik deze coördinaten gebruikt met de GraphHopper Routing API om een route te berekenen.

De applicatie werd vervolgens uitgebreid met:

gebruikersinvoer;

error handling;

afstandsconversie;

reistijdberekening;

route-instructies;

ondersteuning voor car, bike en foot;

automatische fallback naar car.

De uiteindelijke applicatie kan daardoor zelfstandig locaties verwerken, routes opvragen, JSON-data analyseren en de resultaten leesbaar aan de gebruiker weergeven.