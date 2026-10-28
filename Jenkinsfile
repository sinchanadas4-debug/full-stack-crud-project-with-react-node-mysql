pipeline {
  agent any

  environment {
    DOCKERHUB = credentials('dockerhub-creds')
    APP_HOST  = 'ubuntu@15.252.76.26'
    APP_IP    = '15.252.76.26'
    TAG       = "${BUILD_NUMBER}"
    APP_DIR   = 'run_mern_stack_with_docker_compose'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Images') {
      steps {
        sh 'docker build -t $DOCKERHUB_USR/mern-backend:$TAG  $APP_DIR/backend'
        sh 'docker build -t $DOCKERHUB_USR/mern-frontend:$TAG $APP_DIR/frontend'
      }
    }
    stage('Push to Docker Hub') {
      steps {
        sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
        sh 'docker push $DOCKERHUB_USR/mern-backend:$TAG'
        sh 'docker push $DOCKERHUB_USR/mern-frontend:$TAG'
      }
    }
    stage('Deploy to AWS') 
    {
      steps
       {
        sshagent(['app-server-ssh']) {
          sh '''
            scp -o StrictHostKeyChecking=no docker-compose.prod.yml $APP_HOST:~/app/docker-compose.yml
            ssh -o StrictHostKeyChecking=no $APP_HOST "cd ~/app && export DOCKERHUB_USER=$DOCKERHUB_USR IMAGE_TAG=$TAG VITE_API_URL=http://$APP_IP:3000 && docker compose pull && docker compose up -d"
          '''
        }
      }
    }
    stage('Health Check') 
    {
      steps {
        sh 'sleep 30 && curl -f http://$APP_IP:3000/ && curl -f http://$APP_IP/'
      }
    }
  }

  post {
    always { sh 'docker logout || true' }
  }
}