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

        stage('Cleanup Docker') {
            steps {
                sh 'docker system prune -f'
            }
        }

        stage('Deploy to Server') {
            steps {
                withCredentials([
                    string(credentialsId: 'MONGO_URI', variable: 'MONGO_URI'),
                    string(credentialsId: 'JWT_SECRET', variable: 'JWT_SECRET')
                ]) {
                    sh """
                    ssh $APP_SERVER '
                    docker pull $DOCKER_IMAGE:$VERSION
                    docker stop fold-backend || true
                    docker rm fold-backend || true
                    docker run -d -p 8000:8000 \
                    -e MONGO_URI="${MONGO_URI}" \
                    -e JWT_SECRET="${JWT_SECRET}" \
                    --name fold-service \
                    $DOCKER_IMAGE:$VERSION
                    '
                    """
                }
            }
        }
    }
}