pipeline {
    agent any

    environment {
        DOCKER_URL = "docker.io/shouvickp"
        APP_SERVER = "ubuntu@13.126.188.229"
        DOCKER_IMAGE = "shouvickp/fold-backend"
        VERSION = "latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/shouvickp/fold-service.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$VERSION .'
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    withCredentials ([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sh '''
                        echo $ARTIFACTORY_PASSWORD | docker login -u ${ARTIFACTORY_USER} --password-stdin
                        docker push $DOCKER_IMAGE:$VERSION
                        '''
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                sh """
                ssh $APP_SERVER '
                docker pull $DOCKER_IMAGE:$VERSION
                docker stop fold-service || true
                docker rm fold-service || true
                docker run -d -p 5000:5000 --name fold-service $DOCKER_IMAGE:$VERSION
                '
                """
            }
        }
    }
}