pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                // Oude containers en volumes netjes stoppen en verwijderen
                sh 'docker compose down -v || true'
            }
        }

        stage('Build') {
            steps {
                // Containers bouwen en op de achtergrond starten conform de README
                sh 'docker compose up -d --build'
            }
        }

        stage('Results') {
            steps {
                // Wacht even zodat MariaDB en de webapp volledig geïnitialiseerd zijn
                sleep time: 10, unit: 'SECONDS'
                // Controleer of de applicatie reageert via de Docker host gateway
                sh 'curl -f http://172.17.0.1:8080/ || curl -f http://localhost:8080/'
            }
        }
    }
}