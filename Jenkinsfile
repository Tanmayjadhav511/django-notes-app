pipeline {
    agent any

    environment {
        SEMGREP_APP_TOKEN = credentials('semgrep-token')
    }

    stages {

        stage("Clone Code") {
            steps {
                git url: "https://github.com/LondheShubham153/django-notes-app.git", branch: "main"
            }
        }

        stage("Semgrep Scan") {
            steps {
                sh '''
                    pip install semgrep
                    semgrep ci
                '''
            }
        }

        stage("Build and Test") {
            steps {
                sh "docker build . -t note-app-test-new"
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId:"dockerHub",
                    passwordVariable:"dockerHubPass",
                    usernameVariable:"dockerHubUser")]) {

                    sh "docker tag note-app-test-new ${dockerHubUser}/note-app-test-new:latest"
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPass}"
                    sh "docker push ${dockerHubUser}/note-app-test-new:latest"
                }
            }
        }

        stage("Deploy") {
            steps {
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
