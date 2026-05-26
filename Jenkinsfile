pipeline {
    agent any

    stages {
        stage('Source') {
            steps {
                echo 'Hello Dominik - Code wird erfolgreich aus Git geladen...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Hello Dominik - Build-Schritt erfolgreich abgeschlossen.'
            }
        }
        stage('Test') {
            steps {
                echo 'Hello Dominik - Test-Schritt erfolgreich abgeschlossen.'
            }
        }
        stage('Deployment') {
            steps {
                echo 'Hello Dominik - Deployment erfolgreich abgeschlossen.'
            }
        }
    }
}
