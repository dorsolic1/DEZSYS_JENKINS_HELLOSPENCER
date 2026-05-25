pipeline {
    agent any

    stages {
        stage('Source') {
            steps {
                echo 'HelloWorld - Code wird erfolgreich aus Git geladen...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'HelloWorld - Build-Schritt erfolgreich abgeschlossen.'
            }
        }
        stage('Test') {
            steps {
                echo 'HelloWorld - Test-Schritt erfolgreich abgeschlossen.'
            }
        }
        stage('Deployment') {
            steps {
                echo 'HelloWorld - Deployment erfolgreich abgeschlossen.'
            }
        }
    }
}
