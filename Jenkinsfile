pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                // Oude containers en volumes netjes stoppen en verwijderen
                sh 'docker rm -f todoapp todoappdb'
            }
        }

        stage('Build') {
            steps {
                // Containers bouwen en op de achtergrond starten conform de README
                sh '''
                    docker network create todo-net 2>/dev/null || true
                    docker run -d --name todoappdb --network todo-net -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz mariadb:11
                    sleep 15
                    docker build -t todoapp ./TodoApp
                    docker run -d --name todoapp --network todo-net -p 5000:8080 -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" todoapp
                '''
            }
        }

        stage('Results') {
            steps {
                // Wacht even zodat MariaDB en de webapp volledig geïnitialiseerd zijn
                sleep time: 10, unit: 'SECONDS'
                // Controleer of de applicatie reageert via de Docker host gateway
                sh 'curl -f http://172.17.0.1:5000/ || curl -f http://localhost:5000/'
            }
        }
    }
}