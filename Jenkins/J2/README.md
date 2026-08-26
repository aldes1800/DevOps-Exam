Jenkins Pipeline Experiment

Doel

In dit experiment maak ik een eenvoudige Jenkins Pipeline met drie stages:

Build
↓
Test
↓
Deploy

De bestanden worden in Visual Studio Code gemaakt en getest. De echte pipeline wordt daarna door Jenkins uitgevoerd.

Bestanden

Pipeline-Experiment/
├── app.py
└── Jenkinsfile

app.py

print("Hello from my DevOps pipeline!")

Jenkinsfile

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing application...'
                sh 'python3 app.py'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Application deployed successfully!'
            }
        }
    }
}

Testen in Visual Studio Code

Open de terminal in VS Code:

Terminal → New Terminal

Test eerst de Python-code:

python3 -m py_compile app.py
python3 app.py

Verwachte output:

Hello from my DevOps pipeline!

Wat doet de pipeline?

Build

python3 -m py_compile app.py

Controleert of de Python-code geldig is.

Test

python3 app.py

Voert de applicatie uit.

Deploy

Application deployed successfully!

Simuleert een succesvolle deployment.

Wat demonstreer ik?

VS Code
↓
app.py + Jenkinsfile
↓
Build
↓
Test
↓
Deploy

Mondelinge uitleg

Ik heb een Jenkins Pipeline gemaakt met drie stages. In Build controleer ik of de Python-code geldig is, in Test voer ik de applicatie uit en in Deploy simuleer ik een succesvolle deployment. Ik schrijf en test de bestanden in Visual Studio Code, waarna Jenkins de pipeline uitvoert.