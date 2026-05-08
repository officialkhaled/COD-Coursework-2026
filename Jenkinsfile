pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME = "khaled197/portfolio-app:${BUILD_NUMBER}"
        LATEST_IMAGE = "khaled197/portfolio-app:latest"
        WORKER_IP = "107.23.229.88"
    }

    stages {
        stage('Code Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/officialkhaled/COD-Coursework-2026.git'
            }
        }

        stage('Image Build') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME -t $LATEST_IMAGE .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE_NAME
                        docker push $LATEST_IMAGE
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                ssh -i /var/lib/jenkins/.ssh/id_rsa -o StrictHostKeyChecking=no ec2-user@$WORKER_IP '
                    export JENKINS_NODE_COOKIE=dontKillMe &&

                    cd ~/App &&

                    git pull origin main &&

                    kubectl set image deployment/portfolio-deployment portfolio-container=$IMAGE_NAME &&

                    kubectl apply -f k8s/service.yaml &&

                    kubectl rollout status deployment/portfolio-deployment &&

                    pkill -f "kubectl port-forward" || true &&

                    nohup kubectl port-forward --address 0.0.0.0 service/portfolio-service 30007:80 > portforward.log 2>&1 &
                '
                """
            }
        }
    }
}
