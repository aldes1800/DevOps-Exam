REST API Experiment — cURL

Doel

In dit experiment test ik een REST API via de terminal met curl.

Ik gebruik:

GET om data op te halen;

POST om data te versturen;

DELETE om een resource te verwijderen.

De API die ik gebruik is:

https://jsonplaceholder.typicode.com

1. GET request

Data ophalen:

curl https://jsonplaceholder.typicode.com/posts/1

De API geeft een JSON-response terug met informatie over post 1.

2. POST request

Nieuwe data versturen:

curl -X POST https://jsonplaceholder.typicode.com/posts -H "Content-Type: application/json" -d '{"title":"DevOps Lab","body":"My REST API experiment","userId":1}'

Hierbij:

-X POST kiest de HTTP-methode;

-H voegt een HTTP-header toe;

-d stuurt JSON-data mee.

3. DELETE request

Een resource verwijderen:

curl -X DELETE https://jsonplaceholder.typicode.com/posts/1

Hoe werkt het?

curl
↓
HTTP request
↓
REST API
↓
JSON response

Belangrijke begrippen

GET

Haalt gegevens op.

POST

Stuurt nieuwe gegevens naar de API.

DELETE

Verwijdert een resource.

JSON

Formaat waarin de API gegevens terugstuurt.

cURL

Command-line tool om HTTP-requests naar een API te sturen.

Mondelinge uitleg

Ik heb met cURL een REST API getest vanuit de terminal. Ik gebruikte GET om data op te halen, POST om JSON-data te versturen en DELETE om een resource te verwijderen. De API gaf de resultaten terug in JSON-formaat.