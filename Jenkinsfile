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

                    until docker exec todoappdb mariadb -utodo_usr -pletmeinplz -e "SELECT 1;" >/dev/null 2>&1; do
                        echo "Wachten op MariaDB..."
                        sleep 2
                    done

                    docker exec -i todoappdb mariadb -utodo_usr -pletmeinplz todo_db < TodoApp/schema.sql

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

        stage('Deploy Production') {
            steps {
                sshagent(['dmz010-deploy-key']) {
                    sh '''
                        docker save todoapp | gzip | ssh -o StrictHostKeyChecking=no -o BatchMode=yes -o ServerAliveInterval=5 -o ServerAliveCountMax=3 deploy@192.0.2.10 'gunzip | docker load'
                        ssh -o StrictHostKeyChecking=no -o BatchMode=yes deploy@192.0.2.10 '
                            docker rm -f todoapp-prod || true
                            docker run -d --name todoapp-prod --restart unless-stopped -p 8080:8080 \
                              -e ConnectionStrings__TodoDb="Server=192.0.2.20;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" \
                              todoapp
                        '
                    '''
                }
            }
        }
    }
}
