pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                sh 'docker rm -f todoapp todoappdb || true'
                sh 'docker network rm todo-net || true'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker network create todo-net
                    docker run -d --name todoappdb --network todo-net \
                      -e MARIADB_ROOT_PASSWORD=sekrit \
                      -e MARIADB_DATABASE=todo_db \
                      -e MARIADB_USER=todo_usr \
                      -e MARIADB_PASSWORD=letmeinplz \
                      mariadb:11

                    sleep 20

                    docker cp TodoApp/schema.sql todoappdb:/schema.sql
                    docker exec todoappdb sh -c "mariadb -utodo_usr -pletmeinplz todo_db < /schema.sql"

                    docker build -t todoapp ./TodoApp
                    docker run -d --name todoapp --network todo-net -p 5000:8080 \
                      -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" \
                      todoapp
                '''
            }
        }

        stage('Results') {
            steps {
                sleep time: 10, unit: 'SECONDS'
                sh 'curl -f http://172.17.0.1:5000/'
            }
        }
    }
}