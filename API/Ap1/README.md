# Ap1 — Explore REST APIs with API Simulator and Postman

## Doel van de lab

In deze lab heb ik geleerd hoe ik een **REST API** kan gebruiken en testen via verschillende methodes.

Ik heb gewerkt met:

* REST API endpoints
* HTTP-methodes `GET`, `POST` en `DELETE`
* JSON
* HTTP-statuscodes
* Query parameters
* API-authenticatie
* API tokens / API keys
* cURL
* Postman
* Python `requests`
* Python `Faker`

De lab maakt gebruik van de **School Library API Simulator** in de Cisco DEVASC VM. De simulator bevat een database met boeken waarop API-requests kunnen worden uitgevoerd.

---

# 1. DEVASC VM

Voor deze lab heb ik de Cisco **DEVASC Virtual Machine** gebruikt.

De belangrijkste programma's die ik heb gebruikt zijn:

* Chromium
* Terminal
* Postman
* Visual Studio Code
* Python

De School Library applicatie is binnen de VM bereikbaar via:

```text
http://library.demo.local
```

De API-documentatie is bereikbaar via:

```text
http://library.demo.local/api/v1/docs
```

---

# 2. API-documentatie bekijken

Via Chromium heb ik de School Library API-documentatie geopend.

Hierin zijn verschillende endpoints beschikbaar, zoals:

```text
GET    /books
POST   /books
GET    /books/{id}
DELETE /books/{id}
POST   /loginViaBasic
```

De documentatie gebruikt de **OpenAPI Specification** om onder andere requests, responses, headers en parameters te beschrijven.

![School Library API](screenshots/01-api-docs.png)

---

# 3. GET /books

Mijn eerste API-call was:

```http
GET /api/v1/books
```

Hiermee vraag ik alle boeken op die aanwezig zijn in de School Library database.

De request werd verstuurd naar:

```text
http://library.demo.local/api/v1/books
```

De server gaf:

```text
HTTP 200
```

terug.

Statuscode `200` betekent dat de request succesvol werd verwerkt.

De gegevens werden als **JSON** teruggestuurd.

Voorbeeld:

```json
{
  "id": 0,
  "title": "IP Routing Fundamentals",
  "author": "Mark A. Sportack"
}
```

![GET books](screenshots/02-get-books.png)

---

# 4. GET request via cURL

Dezelfde API-call heb ik daarna via de terminal uitgevoerd met `curl`.

```bash
curl -X GET "http://library.demo.local/api/v1/books" -H "accept: application/json"
```

`curl` is een command-line tool waarmee HTTP-requests kunnen worden uitgevoerd.

De terminal gaf dezelfde boekenlijst terug als de API-documentatie.

![cURL GET](screenshots/03-curl-get-books.png)

---

# 5. Query parameter — includeISBN

Daarna heb ik een **query parameter** gebruikt:

```text
includeISBN=true
```

Hierdoor veranderde de request naar:

```text
http://library.demo.local/api/v1/books?includeISBN=true
```

De API gaf nu naast `id`, `title` en `author` ook ISBN-nummers terug.

Voorbeeld:

```json
{
  "id": 0,
  "title": "IP Routing Fundamentals",
  "author": "Mark A. Sportack",
  "isbn": "978-1578700714"
}
```

Hieruit heb ik geleerd dat query parameters gebruikt kunnen worden om het resultaat van een API-request te beïnvloeden.

---

# 6. Authenticatie

Sommige endpoints zijn beveiligd en vereisen authenticatie.

Ik heb hiervoor gebruikt:

```http
POST /api/v1/loginViaBasic
```

Na succesvolle authenticatie kreeg ik van de server een token terug.

Voorbeeld:

```json
{
  "token": "cisco|..."
}
```

Deze token wordt daarna gebruikt als:

```text
X-API-KEY
```

voor beveiligde API-calls.

> **Security:** API-tokens en wachtwoorden worden niet in deze GitHub-repository gepubliceerd.

---

# 7. Boek toevoegen met POST

Om een nieuw boek toe te voegen heb ik gebruikt:

```http
POST /api/v1/books
```

De gegevens werden als JSON in de **request body** verstuurd.

```json
{
  "id": 4,
  "title": "IPv6 Fundamentals",
  "author": "Rick Graziani"
}
```

De request werd succesvol verwerkt met:

```text
HTTP 200
```

![POST book](screenshots/04-post-book-success.png)

Daarna heb ik een tweede boek toegevoegd:

```json
{
  "id": 5,
  "title": "31 Days Before Your CCNA Exam",
  "author": "Allan Johnson"
}
```

Ook deze request gaf statuscode `200`.

![Second POST](screenshots/05-post-second-book.png)

---

# 8. Toegevoegde boeken controleren

Ik heb opnieuw:

```http
GET /api/v1/books
```

uitgevoerd.

Hiermee kon ik controleren dat de twee nieuwe boeken daadwerkelijk in de database aanwezig waren.

![Verify books](screenshots/06-verify-added-books.png)

---

# 9. Eén specifiek boek opvragen

Om één specifiek boek op te vragen gebruikte ik:

```http
GET /api/v1/books/4
```

De `4` is hier het ID van het boek.

De server gaf:

```json
{
  "id": 4,
  "title": "IPv6 Fundamentals",
  "author": "Rick Graziani"
}
```

terug met statuscode `200`.

![Specific book](screenshots/07-get-specific-book.png)

Het verschil is dus:

```text
GET /books
```

→ alle boeken ophalen

```text
GET /books/4
```

→ alleen het boek met ID `4` ophalen

---

# 10. Boek verwijderen

Vervolgens heb ik boek `4` verwijderd met:

```http
DELETE /api/v1/books/4
```

De server gaf opnieuw:

```text
HTTP 200
```

terug en toonde het verwijderde boek in de response.

Daarna heb ik opnieuw `GET /books` uitgevoerd om te controleren dat boek `4` niet langer aanwezig was.

Boek `5` bleef wel aanwezig.

![Verify delete](screenshots/08-verify-delete.png)

---

# 11. REST API testen met Postman

Daarna heb ik dezelfde School Library API gebruikt via **Postman**.

Postman maakt het mogelijk om API-requests eenvoudig te maken, testen, organiseren en opnieuw uit te voeren.

Mijn eerste Postman-request was:

```http
GET http://library.demo.local/api/v1/books
```

De server antwoordde met:

```text
200 OK
```

en gaf de boeken terug als JSON.

![Postman GET](screenshots/09-postman-get-books.png)

---

# 12. Authenticatie in Postman

Voor beveiligde API-calls heb ik in Postman een nieuwe `POST` request gemaakt naar:

```text
http://library.demo.local/api/v1/loginViaBasic
```

Onder **Authorization** gebruikte ik:

```text
Type: Basic Auth
```

Na het versturen van de request kreeg ik een nieuwe API-token terug.

Deze token heb ik vervolgens gebruikt als:

```text
X-API-KEY
```

voor beveiligde requests.

---

# 13. POST request met Postman

In Postman heb ik vervolgens een boek toegevoegd via:

```http
POST http://library.demo.local/api/v1/books
```

Onder **Authorization** gebruikte ik:

```text
Type: API Key
Key: X-API-KEY
Value: [API TOKEN]
```

Voor de Body gebruikte ik:

```text
raw → JSON
```

met:

```json
{
  "id": 4,
  "title": "IPv6 Fundamentals",
  "author": "Rick Graziani",
  "isbn": "978 158144778"
}
```

De server antwoordde met:

```text
200 OK
```

![Postman POST](screenshots/10-postman-post-book-success.png)

---

# 14. Probleem opgelost — 400 Bad Request

Tijdens de POST-request in Postman kreeg ik eerst:

```text
400 BAD REQUEST
```

met:

```text
Input payload validation failed
```

De JSON-inhoud zelf was correct, maar Postman verstuurde de body aanvankelijk niet op de correcte manier.

Ik heb dit opgelost door bij **Body** expliciet te kiezen voor:

```text
raw
```

en vervolgens:

```text
JSON
```

Hierdoor werd de request correct als:

```text
Content-Type: application/json
```

verstuurd.

Na deze aanpassing kreeg ik:

```text
200 OK
```

Dit heeft mij geholpen om het belang van het correcte **Content-Type** bij API-requests beter te begrijpen.

---

# 15. Query parameters in Postman

Ik heb daarna twee query parameters toegevoegd via het tabblad **Params**:

```text
includeISBN = true
sortBy      = author
```

Postman bouwde automatisch deze URL:

```text
http://library.demo.local/api/v1/books?includeISBN=true&sortBy=author
```

Hierdoor:

* werden ISBN-nummers meegegeven;
* werden de boeken gesorteerd op auteur.

![Postman parameters](screenshots/11-postman-query-parameters.png)

---

# 16. API automatiseren met Python

In het laatste deel van de lab heb ik Python gebruikt om automatisch boeken toe te voegen aan de API.

Hiervoor heb ik het script geopend:

```text
add100RandomBooks.py
```

Het script importeert:

```python
import requests
import json
from faker import Faker
```

`requests` wordt gebruikt om HTTP/API requests uit te voeren.

`json` wordt gebruikt voor JSON-data.

`Faker` wordt gebruikt om willekeurige gegevens te genereren.

---

# 17. Faker testen

Ik heb eerst de Faker-library getest.

```python
from faker import Faker

fake = Faker()
```

Voor een willekeurige naam:

```python
print(fake.name())
```

Voor een willekeurige titel:

```python
print(fake.catch_phrase())
```

Voor een willekeurig ISBN:

```python
print(fake.isbn13())
```

Faker genereert telkens andere testgegevens.

---

# 18. Authenticatie via Python

Het Python-script bevat:

```python
APIHOST = "http://library.demo.local"
LOGIN = "cisco"
PASSWORD = "Cisco123!"
```

De functie:

```python
getAuthToken()
```

gebruikt:

```python
requests.post()
```

om automatisch een authenticatierequest naar:

```text
/api/v1/loginViaBasic
```

te sturen.

Bij statuscode `200` haalt het programma de token uit de JSON-response.

---

# 19. Boeken toevoegen via Python

De functie:

```python
addBook(book, apiKey)
```

voert een POST-request uit naar:

```text
/api/v1/books
```

De API-key wordt meegestuurd in de header:

```text
X-API-Key
```

en het boek wordt als JSON verstuurd.

Het script gebruikt Faker voor:

```python
fakeTitle = fake.catch_phrase()
fakeAuthor = fake.name()
fakeISBN = fake.isbn13()
```

en maakt daarmee een object:

```python
book = {
    "id": i,
    "title": fakeTitle,
    "author": fakeAuthor,
    "isbn": fakeISBN
}
```

---

# 20. 100 boeken automatisch toevoegen

Het volledige script heb ik uitgevoerd met:

```bash
python3 add100RandomBooks.py
```

De terminal bevestigde voor ieder boek dat het succesvol werd toegevoegd.

Voorbeeld:

```text
Book {'id': 95, ...} added.
Book {'id': 96, ...} added.
Book {'id': 97, ...} added.
```

![Python 100 books](screenshots/12-python-add-100-books.png)

Daarna heb ik opnieuw de School Library API gecontroleerd en zag ik de automatisch gegenereerde boeken terug.

![Verify Python books](screenshots/13-python-books-verified.png)

Het lab gebruikt Python en Faker specifiek om het handmatig toevoegen van grote aantallen boeken te automatiseren.

---

# 21. Pagination

Na het toevoegen van een groot aantal boeken worden niet alle resultaten tegelijkertijd weergegeven.

De API ondersteunt hiervoor de parameter:

```text
page
```

Bijvoorbeeld:

```text
/api/v1/books?page=2
```

Hiermee kunnen verschillende pagina's van de boekenlijst worden opgevraagd. De lab vermeldt dat via de `page` parameter de overige boeken kunnen worden bekeken.

---

# 22. Nog 100 boeken toevoegen

Om opnieuw 100 boeken toe te voegen moeten nieuwe, nog niet gebruikte IDs worden gebruikt.

Als de huidige IDs bijvoorbeeld tot `104` lopen, kan het bereik worden aangepast naar:

```python
for i in range(105, 205):
```

Dit genereert 100 nieuwe IDs van `105` tot en met `204`.

---

# Problemen en oplossingen

## 1. Authenticatie

Tijdens het uitvoeren van de lab moest ik ervoor zorgen dat de juiste credentials en API-token werden gebruikt.

Ik heb geleerd dat beveiligde endpoints een geldige:

```text
X-API-KEY
```

vereisen.

## 2. 400 Bad Request in Postman

Mijn POST-request gaf aanvankelijk:

```text
400 BAD REQUEST
```

De oplossing was de request body correct instellen op:

```text
Body → raw → JSON
```

Daarna werkte dezelfde request wel en kreeg ik:

```text
200 OK
```

## 3. Python interpreter

Tijdens het testen van Faker voerde ik:

```text
python3
```

in terwijl ik al in de Python-interpreter zat.

Daardoor kreeg ik:

```text
NameError: name 'python3' is not defined
```

Ik heb geleerd dat:

```text
devasc@labvm:~$
```

de Linux shell is, terwijl:

```text
>>>
```

betekent dat ik al in de Python-interpreter zit.

De Python-interpreter kan worden afgesloten met:

```python
quit()
```

---

# Wat heb ik geleerd?

Na deze lab begrijp ik:

* hoe een REST API werkt;
* wat een API endpoint is;
* waarvoor `GET`, `POST` en `DELETE` worden gebruikt;
* hoe JSON wordt gebruikt om gegevens uit te wisselen;
* wat HTTP-statuscode `200` betekent;
* wat een `400 Bad Request` betekent;
* hoe query parameters werken;
* hoe API-authenticatie met tokens werkt;
* hoe een `X-API-KEY` header wordt gebruikt;
* hoe API-calls met cURL worden uitgevoerd;
* hoe Postman wordt gebruikt om API's te testen;
* hoe Python `requests` gebruikt voor HTTP-requests;
* hoe Faker testdata kan genereren;
* hoe API-taken met Python geautomatiseerd kunnen worden;
* hoe pagination gebruikt wordt bij grote hoeveelheden resultaten.

---

# Belangrijkste commando's

## GET via cURL

```bash
curl -X GET "http://library.demo.local/api/v1/books" -H "accept: application/json"
```

## Python starten

```bash
python3
```

## Python verlaten

```python
quit()
```

## Python-script uitvoeren

```bash
python3 add100RandomBooks.py
```

---

# Belangrijkste endpoints

| Methode  | Endpoint                | Functie                      |
| -------- | ----------------------- | ---------------------------- |
| `GET`    | `/api/v1/books`         | Alle boeken ophalen          |
| `GET`    | `/api/v1/books/{id}`    | Eén boek ophalen             |
| `POST`   | `/api/v1/books`         | Een boek toevoegen           |
| `DELETE` | `/api/v1/books/{id}`    | Een boek verwijderen         |
| `POST`   | `/api/v1/loginViaBasic` | Authenticatietoken aanvragen |

---

# Conclusie

In deze lab heb ik eerst REST API-calls handmatig uitgevoerd via de School Library API-documentatie en cURL.

Daarna heb ik dezelfde API getest met **Postman**, inclusief authenticatie, API keys, JSON bodies en query parameters.

Ten slotte heb ik met **Python**, `requests` en `Faker` het proces geautomatiseerd en een groot aantal boeken automatisch via de REST API toegevoegd.

Hierdoor begrijp ik niet alleen hoe ik een REST API kan gebruiken, maar ook hoe dezelfde API-acties via verschillende tools kunnen worden uitgevoerd en uiteindelijk met code kunnen worden geautomatiseerd.
