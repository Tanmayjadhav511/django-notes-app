pipeline {
    agent any

    stages {

        stage("Semgrep Scan") {
            agent {
                docker {
                    image 'python:3.11'
                    args '-u root'   // ensures permissions
                }
            }
            steps {
                withCredentials([string(credentialsId: 'semgrep-token', variable: 'SEMGREP_APP_TOKEN')]) {
                    sh '''
                        pip install --quiet semgrep
                        semgrep ci
                    '''
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                sh '''
                    docker build -t note-app-test-new .
                '''
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHub",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh '''
                        docker tag note-app-test-new ${DOCKER_USER}/note-app-test-new:latest
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${DOCKER_USER}/note-app-test-new:latest
                    '''
                }
            }
        }

        stage("Deploy") {
            steps {
                sh '''
                    docker-compose down || true
                    docker-compose up -d
                '''
            }
        }
    }
}
