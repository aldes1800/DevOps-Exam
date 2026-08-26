REST API Experiment — cURL Forms

Doel

In dit experiment verstuur ik formuliergegevens naar een REST API met curl.

Ik toon twee manieren:

-d voor normale form data;

-F voor multipart/form-data.

1. Form data versturen met -d

curl -X POST https://httpbin.org/post -d "name=Alan" -d "course=DevOps"

De API geeft de ontvangen gegevens terug in JSON.

Voorbeeld:

"form": {
  "course": "DevOps",
  "name": "Alan"
}

-d verstuurt de gegevens als:

application/x-www-form-urlencoded

2. Multipart form versturen met -F

curl -X POST https://httpbin.org/post -F "name=Alan" -F "course=DevOps"

-F verstuurt de gegevens als:

multipart/form-data

Verschil tussen -d en -F

-d  → application/x-www-form-urlencoded
-F  → multipart/form-data

-F wordt vaak gebruikt wanneer een formulier ook bestanden moet kunnen versturen.

Hoe werkt het?

cURL
↓
POST request
↓
Form data
↓
REST API
↓
JSON response

Mondelinge uitleg

Ik heb met cURL formuliergegevens naar een REST API gestuurd. Met -d verstuur ik gewone form fields en met -F multipart form data. De API ontvangt de gegevens en geeft ze terug in JSON-formaat.