Docker Management Experiment

Doel

In dit experiment beheer ik Docker containers en images met enkele basiscommando's.

Containers bekijken

Alle actieve containers:

docker ps

Alle containers, inclusief gestopte:

docker ps -a

Images bekijken

docker images

Container stoppen

docker stop mynginx

Container verwijderen

docker rm mynginx

Image verwijderen

docker rmi nginx

Docker opruimen

docker system prune

Dit verwijdert ongebruikte Docker-resources.

Hoe werkt het?

Docker Management
↓
Containers bekijken
↓
Images bekijken
↓
Stoppen
↓
Verwijderen
↓
Opruimen

Mondelinge uitleg

Ik heb Docker management-commando's gebruikt om containers en images te bekijken, containers te stoppen en verwijderen en ongebruikte Docker-resources op te ruimen.