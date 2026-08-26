Ansible Playbook Experiment — Webserver met Custom Homepage

Doel

In dit experiment gebruik ik een Ansible playbook om automatisch:

Apache te installeren;

Apache te starten;

een eigen index.html te plaatsen.

Playbook

Bestand:

webserver.yml

---
- name: Configure webserver
  hosts: localhost
  become: yes

  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Create custom homepage
      copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html>
          <head>
              <title>DevOps Webserver</title>
          </head>
          <body>
              <h1>Welcome to my Ansible Webserver!</h1>
              <p>This page was deployed automatically with Ansible.</p>
          </body>
          </html>

Uitvoeren

ansible-playbook webserver.yml --ask-become-pass

Resultaat:

ok=4
changed=1
failed=0

Dit betekent dat het playbook succesvol uitgevoerd werd.

Webserver testen

In mijn omgeving luistert Apache op poort 8081.

curl http://localhost:8081

De eigen homepage wordt dan teruggegeven.

Probleem tijdens het testen

Met:

curl http://localhost

kreeg ik:

Failed to connect to localhost port 80: Connection refused

De oorzaak was dat Apache niet op poort 80, maar op poort 8081 luisterde.

Controle:

grep -R "Listen" /etc/apache2/ports.conf

Daarna werkte:

curl http://localhost:8081

Wat demonstreer ik?

Ansible Playbook
↓
Apache installeren
↓
Apache starten
↓
Custom index.html plaatsen
↓
Webserver testen

Mondelinge uitleg

Ik heb een Ansible-playbook gemaakt dat Apache automatisch installeert en start. Daarna gebruikt het playbook de copy module om een eigen homepage in /var/www/html/index.html te plaatsen. De webserver luistert in mijn omgeving op poort 8081 en werd getest met curl.