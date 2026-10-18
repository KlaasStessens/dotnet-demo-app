pipeline {
    agent any
    stages {
        stage('Preparation') {
            steps {
                sh 'docker rm -f todoappdb todoapp testdb || true'
            }
        }
        stage('Unit tests') {
            steps {
                sh '''
                    docker create --name testdb \
                      -e MARIADB_ROOT_PASSWORD=sekrit \
                      -e MARIADB_DATABASE=todo_test_db \
                      -e MARIADB_USER=todo_usr \
                      -e MARIADB_PASSWORD=letmeinplz \
                      mariadb:11
                    docker cp TodoApp/schema.sql testdb:/docker-entrypoint-initdb.d/schema.sql
                    docker start testdb

                    for i in $(seq 1 30); do
                        docker exec testdb mariadb -h127.0.0.1 -utodo_usr -pletmeinplz todo_test_db \
                          -e 'SELECT 1 FROM todos LIMIT 1' >/dev/null 2>&1 && break
                        sleep 2
                    done

                    docker run --rm --volumes-from jenkins_server \
                      --network container:testdb \
                      -w "$WORKSPACE" \
                      mcr.microsoft.com/dotnet/sdk:10.0 dotnet test
                '''
            }
            post {
                always { sh 'docker rm -f testdb || true' }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker compose up -d --build'
            }
        }
        stage('Smoke test') {
            steps {
                sh '''
                    for i in $(seq 1 20); do
                        curl -fsS http://172.16.0.10:8081/ && exit 0
                        sleep 3
                    done
                    exit 1
                '''
            }
        }
    }
}