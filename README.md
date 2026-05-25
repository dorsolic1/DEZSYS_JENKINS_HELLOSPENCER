# Jenkins - GK8.3
Dominik Orsolic

## Schritt 1: Docker Container
Mit folgendem Befehl einen Docker Container erstellen:
```bash
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-einfach jenkins/jenkins:lts
```

Um zu checken ob dieser läuft:
```bash
docker ps
```

`localhost:8080` besuchen
<img width="1487" height="936" alt="grafik" src="https://github.com/user-attachments/assets/ed003d42-51ff-46fb-b2e3-81b72797da77" />


Wenn dieses Fenster erscheint, hat das Starten erfolgreich funktioniert

## Schritt 2: Installation
Da ein Administrator Passwort verlangt wird, muss dieses bei der Ausgabe von `docker logs jenkins-einfach` gefunden werden. Dieses besteht aus einer Kombination von Zahlen und Buchstabe, in meinem Fall: `cfa5cf3b52e54172994cd557cf96d02e`

Auf der Folgeseite auf `Install suggestet plugins klicken`. Danach auf `Skip and continue as Admin`.

Nun ist das Dashboard sichtbar
<img width="1919" height="987" alt="grafik" src="https://github.com/user-attachments/assets/051a4fe2-98a4-42f8-9a49-76c1fb15954f" />



## Schritt 3: Hello World Code
Folgendes Github Repository forken: `https://github.com/ThomasMicheler/DEZSYS_JENKINS_HELLOSPENCER.git`

Den Inhalt der Datei `Jenkinsfile` löschen und durch folgenden Code ersetzten: 
```bash
pipeline {
    agent {
        docker { 
            image 'python:3.11' 
            args '-p 5556:5556'
        }
    }
    environment {
        APP_PORT = '5556'
        GITHUB_REPO = 'https://github.com/ThomasMicheler/DEZSYS_JENKINS_HELLOSPENCER.git'
    }
    stages {
        stage('Pre-Build Cleanup') {
            steps {
                // Kill any existing Flask processes
                sh 'pkill -f "python hello.py" || true'
            }
        }
        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }
        stage('Build') {
            steps {
                sh '''
                    python -m pip install --upgrade pip
                    pip install flask
                    pip install requests
                    pip install pytest
                    if [ ! -f count.txt ]; then
                        echo "0" > count.txt
                    fi
                    chmod 666 count.txt
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                    # Run the unit tests
                    python -m pytest tests/test_hello.py -v
                '''
            }
        }
        stage('Run') {
            steps {
                sh '''
                    nohup python src/hello.py > app.log 2>&1 &
                    sleep 5
                    curl http://localhost:5556/api/hello
                '''
            }
        }
        stage('Test API') {
            steps {
                sh 'python tests/test_api.py'
            }
        }
        stage('Keep Alive') {
            steps {
                // Keep the container running indefinitely
                sh 'sleep infinity'
            }
        }
    }
    post {
        always {
            // Cleanup: Stop the Flask application
            sh 'pkill -f "python src/hello.py" || true'
        }
    }
}
```
Änderungen anschließend commiten.

## Schritt 4: Pipeline
Auf dem Jenkins-Dashboard auf `Element anlegen` klicken, Name eingeben und Typ `Pipeline auswählen`. Anschließend auf `OK` klicken. 

Auf der Konfigurationsseite nach unten zum Bereich `Pipeline` scrollen. Beim Dropdown-Menü `Definition` von `Pipeline script` auf `Pipeline script from SCM` ändern. Anschließend bei `SCM` den Eintrag `Git` auswählen. Jetzt den Link vom geforkten Repository einfügen und `Branch specifier` von `*/master` zu `*/main` ändern 
<img width="1919" height="916" alt="grafik" src="https://github.com/user-attachments/assets/4b6c9ddd-88fc-4203-90de-9d2386fd36f9" />

Auf `Save` klicken

## Schritt 5: Testen
Im Menü auf `Build Now` klicken und warten. Unter `Stages` ist nun zu sehen, dass jede Stage erfolgreich durchgelaufen ist:
<img width="582" height="163" alt="grafik" src="https://github.com/user-attachments/assets/0c8ee141-3c23-40d9-abe9-2e64f51099bb" />


Unter `Console Output` kann man auch die Ausgabe sehen:
<img width="950" height="758" alt="grafik" src="https://github.com/user-attachments/assets/5c97c316-e0a3-4e69-b394-ad08bcfa71ba" />


Hier sind folgende Zeilen sichbar:
`HelloWorld - Code wird erfolgreich aus Git geladen...` -> Beweist die Source-Phase
`HelloWorld - Build-Schritt erfolgreich abgeschlossen` -> Beweist die Build-Phase
`HelloWorld - Test-Schritt erfolgreich abgeschlossen.` -> Beweist die Test-Phase
`HelloWorld - Deployment erfolgreich abgeschlossen` -> Beweist die Deployment-Phase

Ganz unten ist die Zeile `Finished: SUCCESS` zu sehen, was beweist, dass alles Fehlerfrei geklappt hat.

## Quellen
- [Praesentaiton CI/CD](https://elearning.tgm.ac.at/pluginfile.php/251839/mod_resource/content/5/introduction_cicd.pdf)
- [Introduction CI/CD Pipeline](https://semaphore.io/blog/cicd-pipeline).
- [Jenkins – Simple example](https://www.jenkins.io/doc/pipeline/tour/hello-world/#examples).

