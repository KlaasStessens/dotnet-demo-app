pipeline {
    agent any
    stages {
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