pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend image..."
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Deploying backend containers..."
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Deploying nginx load balancer..."
                docker rm -f nginx-lb || true

                # IMPORTANT: USE PORT 8081 because Windows Home + WSL2 blocks 80
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 8081:80 \
                  nginx

                # COPY YOUR CONFIG CORRECTLY (NO SUBFOLDER)
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. Access the app at: http://localhost:8081'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
