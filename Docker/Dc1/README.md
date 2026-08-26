Run Containers Experiment — Nginx

Doel

In dit experiment start ik een bestaande Nginx Docker image als container en test ik de webserver.

Container starten

docker run -d -p 8080:80 --name mynginx nginx

Betekenis:

-d = draait op de achtergrond

-p 8080:80 = hostpoort 8080 naar containerpoort 80

--name mynginx = naam van de container

nginx = gebruikte Docker image

Controleren

docker ps

Hiermee controleer ik of de container actief is.

Webserver testen

curl http://localhost:8080

De standaard Nginx-webpagina wordt teruggegeven.

Container stoppen

docker stop mynginx

Container verwijderen

docker rm mynginx

Hoe werkt het?

Nginx image
↓
docker run
↓
Container
↓
Port mapping
↓
Webserver testen

Mondelinge uitleg

Ik heb een bestaande Nginx Docker image gebruikt om een container te starten. Met port mapping heb ik poort 8080 van de host gekoppeld aan poort 80 van de container. Daarna heb ik de webserver getest, de container gestopt en verwijderd.