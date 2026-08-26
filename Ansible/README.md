Ansible Lab — Automate Installing and Configuring an Apache Web Server

Doel van de lab

In deze lab heb ik Ansible gebruikt om een Apache-webserver automatisch te installeren en configureren op de Cisco DEVASC VM.

De lab bestaat uit vijf delen:

DEVASC VM starten.

Ansible configureren.

Communicatie met de lokale webserver controleren.

Ansible playbooks maken om Apache automatisch te installeren.

Het Apache-playbook uitbreiden zodat de webserver op poort 8081 luistert.

Tijdens de lab heb ik gewerkt met:

Ansible

SSH

YAML

Apache2

Ansible inventory

ansible.cfg

Ansible ad-hoc commands

Ansible playbooks

Ansible modules

handlers

lineinfile

systemctl

1. Omgeving

De lab werd uitgevoerd in de Cisco DEVASC VM.

De lokale webserver gebruikt het dummy-IP-adres:

192.0.2.3

Dit adres bevindt zich op de dummy0 interface van de VM.

De Ansible-bestanden voor deze lab bevinden zich in:

~/labs/devnet-src/ansible/ansible-apache

2. SSH-server starten

Ansible gebruikt SSH om verbinding te maken met hosts.

De SSH-server werd gestart met:

sudo systemctl start ssh

Daarna werd gecontroleerd of SSH actief was:

systemctl status ssh

Resultaat:

Active: active (running)

Hiermee was de VM klaar om Ansible-verbindingen te accepteren.

3. Ansible inventory configureren

In het bestand:

hosts

werd een Ansible-groep met de naam webservers aangemaakt.

Voorbeeld:

[webservers]
192.0.2.3 ansible_ssh_user=devasc ansible_ssh_pass=<PASSWORD>

Betekenis

[webservers] is de naam van de groep waarop Ansible opdrachten kan uitvoeren.

192.0.2.3 is de host waarop de taken worden uitgevoerd.

ansible_ssh_user bepaalt met welke gebruiker Ansible via SSH inlogt.

ansible_ssh_pass bevat het SSH-wachtwoord.

Voor een publieke GitHub-repository publiceer ik geen echte wachtwoorden of andere geheime gegevens.

4. ansible.cfg

Daarna werd het bestand ansible.cfg geconfigureerd.

[defaults]

# Use local hosts file in this folder
inventory=./hosts

# Don't worry about RSA Fingerprints
host_key_checking = False

# Do not create retry files
retry_files_enabled = False

Betekenis

inventory=./hosts

Hiermee weet Ansible welk inventory-bestand gebruikt moet worden.

host_key_checking = False

Hierdoor vraagt Ansible niet om SSH RSA fingerprints handmatig te bevestigen.

retry_files_enabled = False

Hierdoor maakt Ansible geen retry-bestanden aan wanneer een taak mislukt.

5. Communicatie testen met de Ansible ping-module

De verbinding met de groep webservers werd getest met:

ansible webservers -m ping

Resultaat:

192.0.2.3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

Dit toont aan dat Ansible succesvol via SSH met de host kan communiceren.

De Ansible ping-module is niet hetzelfde als een gewone ICMP-ping. Ansible maakt verbinding met de host en controleert of Python daar kan worden uitgevoerd.

6. Communicatie testen met de command-module

Daarna werd getest of Ansible effectief een commando kon uitvoeren op de webserver:

ansible webservers -m command -a "/bin/echo hello world"

Resultaat:

192.0.2.3 | CHANGED | rc=0 >>
hello world

rc=0 betekent dat het Linux-commando succesvol werd uitgevoerd.

7. Eerste Ansible playbook

Daarna werd dezelfde echo-test geautomatiseerd met een YAML-playbook:

test_apache_playbook.yaml

Inhoud:

---
- hosts: webservers
  tasks:
    - name: run echo command
      command: /bin/echo hello world

Het playbook werd uitgevoerd met:

ansible-playbook -v test_apache_playbook.yaml

Resultaat:

PLAY RECAP
192.0.2.3 : ok=2 changed=1 unreachable=0 failed=0

failed=0 betekent dat het playbook zonder fouten werd uitgevoerd.

8. Apache automatisch installeren

Vervolgens werd een nieuw playbook gemaakt:

install_apache_playbook.yaml

Inhoud:

---
- hosts: webservers
  become: yes

  tasks:
    - name: INSTALL APACHE2
      apt:
        name: apache2
        update_cache: yes
        state: latest

    - name: ENABLED MOD_REWRITE
      apache2_module:
        name: rewrite
        state: present
      notify:
        - RESTART APACHE2

  handlers:
    - name: RESTART APACHE2
      service:
        name: apache2
        state: restarted

Het playbook werd uitgevoerd met:

ansible-playbook -v install_apache_playbook.yaml

Resultaat:

ok=4
changed=3
unreachable=0
failed=0

Apache werd dus succesvol geïnstalleerd.

9. Belangrijke onderdelen van het installatie-playbook

hosts: webservers

- hosts: webservers

Hiermee wordt het playbook uitgevoerd op alle hosts in de groep [webservers].

become: yes

become: yes

Hierdoor kan Ansible taken met verhoogde rechten uitvoeren, vergelijkbaar met sudo.

apt

apt:
  name: apache2
  update_cache: yes
  state: latest

De apt-module beheert packages op Debian/Ubuntu-systemen.

apache2_module

apache2_module:
  name: rewrite
  state: present

Hiermee wordt de Apache-module mod_rewrite ingeschakeld.

notify

notify:
  - RESTART APACHE2

Een taak kan hiermee een handler activeren.

Handler

handlers:
  - name: RESTART APACHE2
    service:
      name: apache2
      state: restarted

Een handler wordt alleen uitgevoerd wanneer hij door een taak wordt aangeroepen.

10. Controleren of Apache draait

Na de installatie werd de Apache-service gecontroleerd:

sudo systemctl status apache2

Resultaat:

Active: active (running)

Dit bevestigt dat Apache succesvol werd gestart.

11. Apache in Chromium testen

De webserver werd daarna geopend in Chromium:

http://192.0.2.3

De Apache2 Ubuntu Default Page werd weergegeven.

Hiermee werd bevestigd dat Apache geïnstalleerd is, actief is en HTTP-verkeer op poort 80 werkt.

12. Apache configuratie vóór de wijziging

Voor de laatste stap werd gecontroleerd op welke poort Apache luisterde.

/etc/apache2/ports.conf

cat /etc/apache2/ports.conf

Belangrijk resultaat:

Listen 80

/etc/apache2/sites-available/000-default.conf

cat /etc/apache2/sites-available/000-default.conf

Belangrijk resultaat:

<VirtualHost *:80>

Apache gebruikte dus standaard poort 80.

13. Apache automatisch naar poort 8081 wijzigen

Daarna werd een nieuw playbook gemaakt:

install_apache_options_playbook.yaml

Inhoud:

---
- hosts: webservers
  become: yes

  tasks:
    - name: INSTALL APACHE2
      apt:
        name: apache2
        update_cache: yes
        state: latest

    - name: ENABLED MOD_REWRITE
      apache2_module:
        name: rewrite
        state: present
      notify:
        - RESTART APACHE2

    - name: APACHE2 LISTEN ON PORT 8081
      lineinfile:
        dest: /etc/apache2/ports.conf
        regexp: "^Listen 80"
        line: "Listen 8081"
        state: present
      notify:
        - RESTART APACHE2

    - name: APACHE2 VIRTUALHOST ON PORT 8081
      lineinfile:
        dest: /etc/apache2/sites-available/000-default.conf
        regexp: "^<VirtualHost \\*:80>"
        line: "<VirtualHost *:8081>"
        state: present
      notify:
        - RESTART APACHE2

  handlers:
    - name: RESTART APACHE2
      service:
        name: apache2
        state: restarted

Het playbook werd uitgevoerd met:

ansible-playbook install_apache_options_playbook.yaml

Resultaat:

ok=6
changed=3
unreachable=0
failed=0

14. lineinfile module

De belangrijkste nieuwe module in het laatste playbook was lineinfile.

Deze module kan een specifieke regel in een bestand zoeken en aanpassen.

Voorbeeld:

lineinfile:
  dest: /etc/apache2/ports.conf
  regexp: "^Listen 80"
  line: "Listen 8081"
  state: present

Hiermee zoekt Ansible naar:

Listen 80

en vervangt deze regel door:

Listen 8081

Hetzelfde principe werd gebruikt voor:

<VirtualHost *:80>

dat werd veranderd naar:

<VirtualHost *:8081>

15. Configuratie na het playbook controleren

Na het uitvoeren van het playbook werden de configuratiebestanden opnieuw bekeken.

/etc/apache2/ports.conf

cat /etc/apache2/ports.conf

Resultaat:

Listen 8081

/etc/apache2/sites-available/000-default.conf

cat /etc/apache2/sites-available/000-default.conf

Resultaat:

<VirtualHost *:8081>

Hiermee werd bevestigd dat Ansible de Apache-configuratie correct had aangepast.

16. Apache testen op poort 8081

Als laatste controle werd Chromium geopend op:

http://192.0.2.3:8081

De Apache2 Ubuntu Default Page werd opnieuw succesvol weergegeven.

Hiermee was bewezen dat Apache nu op poort 8081 luistert.

Bestanden van de lab

De belangrijkste bestanden zijn:

ansible-apache/
├── ansible.cfg
├── hosts
├── test_apache_playbook.yaml
├── install_apache_playbook.yaml
└── install_apache_options_playbook.yaml

Aanbevolen GitHub-structuur

Ansible/
└── An1-Apache-Automation/
    ├── README.md
    ├── ansible.cfg
    ├── hosts
    ├── test_apache_playbook.yaml
    ├── install_apache_playbook.yaml
    ├── install_apache_options_playbook.yaml
    └── screenshots/
        ├── 01-ansible-ping-success.png
        ├── 02-ansible-command-success.png
        ├── 03-test-playbook-success.png
        ├── 04-apache-playbook-success.png
        ├── 05-apache-service-running.png
        ├── 06-apache-default-page.png
        ├── 07-apache-port-80-before.png
        ├── 08-apache-port-8081-playbook-success.png
        ├── 09-apache-config-port-8081.png
        └── 10-apache-browser-port-8081.png

Aanbevolen screenshots

ansible webservers -m ping met SUCCESS en pong.

Ansible command-module met hello world.

test_apache_playbook.yaml met succesvolle PLAY RECAP.

Apache-installatieplaybook met failed=0.

systemctl status apache2 met active (running).

Apache2 Ubuntu Default Page op 192.0.2.3.

Configuratie vóór wijziging met Listen 80 en <VirtualHost *:80>.

Playbook dat poort 8081 configureert.

Configuratie na wijziging met Listen 8081 en <VirtualHost *:8081>.

Apache2 Ubuntu Default Page op 192.0.2.3:8081.

Belangrijke begrippen voor het examen

Wat is Ansible?

Ansible is een automation- en configuration-managementtool waarmee systemen automatisch geconfigureerd kunnen worden.

Wat is een inventory?

Een inventory bevat de hosts waarop Ansible opdrachten kan uitvoeren.

Voorbeeld:

[webservers]
192.0.2.3

Wat is een playbook?

Een playbook is een YAML-bestand waarin staat op welke hosts taken uitgevoerd worden, welke taken uitgevoerd worden en welke modules en handlers gebruikt worden.

Wat is YAML?

YAML is een dataformat dat veel wordt gebruikt voor configuratiebestanden. YAML is gevoelig voor correcte inspringing.

Wat is een Ansible module?

Een module voert een bepaalde soort taak uit.

Modules uit deze lab:

ping
command
apt
apache2_module
service
lineinfile

Wat doet de ping module?

ansible webservers -m ping

Controleert of Ansible met de host kan verbinden en daar Python kan uitvoeren.

Wat doet de command module?

De command-module voert een commando uit op een remote host.

Wat betekent become: yes?

become: yes

zorgt ervoor dat Ansible taken met verhoogde rechten kan uitvoeren.

Wat doet de apt module?

De apt module beheert softwarepackages op Debian/Ubuntu.

Wat is een handler?

Een handler is een speciale taak die alleen uitgevoerd wordt wanneer een andere taak hem via notify activeert.

Waarom handlers gebruiken?

Een service hoeft niet na iedere taak opnieuw gestart te worden. Een handler zorgt ervoor dat Apache alleen wordt herstart wanneer een relevante configuratie effectief veranderd is.

Wat doet lineinfile?

lineinfile zoekt naar een regel in een bestand en zorgt dat de gewenste regel aanwezig is.

Wat betekent changed?

changed betekent dat Ansible effectief iets op de host heeft aangepast.

Wat betekent ok?

ok betekent dat een taak succesvol was, maar niet noodzakelijk een wijziging heeft uitgevoerd.

Wat betekent failed=0?

Dit betekent dat geen enkele taak in het playbook is mislukt.

Wat betekent unreachable=0?

Dit betekent dat alle hosts waarop het playbook moest worden uitgevoerd bereikbaar waren.

Problemen en aandachtspunten

YAML indentation

YAML is zeer gevoelig voor spaties.

Correct voorbeeld:

tasks:
  - name: INSTALL APACHE2
    apt:
      name: apache2
      state: latest

SSH moet actief zijn

Ansible kon pas verbinding maken nadat SSH gestart was:

sudo systemctl start ssh

Apache-configuratie moet op twee plaatsen gewijzigd worden

Alleen Listen 8081 instellen is niet voldoende.

Ook:

<VirtualHost *:8081>

moet correct geconfigureerd worden.

Daarom wijzigde het playbook zowel:

/etc/apache2/ports.conf

als:

/etc/apache2/sites-available/000-default.conf

Security

Het inventory-bestand uit de lab gebruikt eenvoudige testcredentials.

Voor een publieke GitHub-repository is het beter geen echte wachtwoorden rechtstreeks in een inventory-bestand te publiceren.

Bijvoorbeeld:

ansible_ssh_pass=<PASSWORD>

Wat heb ik geleerd?

Na deze lab begrijp ik:

wat Ansible is;

hoe Ansible via SSH werkt;

wat een inventory is;

hoe hostgroepen werken;

wat ansible.cfg doet;

hoe Ansible ad-hoc commands werken;

wat een Ansible module is;

hoe een YAML-playbook wordt opgebouwd;

waarom YAML indentation belangrijk is;

hoe ansible-playbook werkt;

hoe Apache automatisch geïnstalleerd kan worden;

wat become: yes doet;

hoe de apt module werkt;

hoe handlers en notify werken;

hoe services automatisch herstart kunnen worden;

hoe lineinfile configuratiebestanden kan aanpassen;

hoe Apache van poort 80 naar 8081 geconfigureerd kan worden.

Mondeling examen — korte uitleg

Als ik deze lab kort moet uitleggen:

In deze lab heb ik Ansible geconfigureerd om via SSH een lokale webserver op 192.0.2.3 te beheren. Eerst heb ik de communicatie getest met de ping- en command-modules. Daarna heb ik met YAML-playbooks Apache2 automatisch geïnstalleerd en mod_rewrite ingeschakeld. Met een handler werd Apache automatisch herstart wanneer een wijziging plaatsvond. Ten slotte heb ik met de lineinfile module de Apache-configuratie aangepast van poort 80 naar poort 8081 en gecontroleerd dat de webserver op die nieuwe poort bereikbaar was.

Conclusie

In deze lab heb ik geleerd hoe Ansible gebruikt kan worden om serverconfiguratie te automatiseren.

Ik begon met het configureren van SSH, het inventory-bestand en ansible.cfg. Daarna controleerde ik de communicatie met Ansible ad-hoc commands.

Vervolgens heb ik playbooks gemaakt om:

opdrachten automatisch uit te voeren;

Apache2 automatisch te installeren;

Apache-modules in te schakelen;

Apache via handlers opnieuw te starten;

configuratiebestanden automatisch aan te passen;

de HTTP-poort van 80 naar 8081 te wijzigen.

Het eindresultaat was een werkende Apache-webserver die volledig met Ansibl