pipeline {
    agent any

    stages {

        stage("Semgrep Scan") {
            steps {
                withCredentials([string(credentialsId: 'semgrep-token', variable: 'SEMGREP_APP_TOKEN')]) {
                    sh '''
                        # Create virtual environment (fix PEP 668 issue)
                        python3 -m venv venv
                        . venv/bin/activate

                        # Install semgrep safely
                        pip install --quiet semgrep

                        # Run scan
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
                    docker-compose down
                    docker-compose up -d
                '''
            }
        }
    }
}
