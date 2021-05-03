pipeline {
  agent any
  environment {
    imageName = 'jaimesalas/math-api:latest'
    ec2Instance = 'ec2-3-8-33-70.eu-west-2.compute.amazonaws.com'
    appPort = 80
  }
  stages {
    stage('Install dependecies') {
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm ci'
      }
    }
    stage('Tests') {
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm test'
      }
    }
    stage('E2E Tets') {
      when {
        branch 'staging'
      }
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true 
        }
      }
      environment {
        BASE_API_URL = "http://$ec2Instance:$appPort"
      }
      steps {
        sh 'npm run test:e2e'
      }
    }
    stage('Build image & push it to DockerHub') {
      when {
        branch 'develop'
      }
      steps {
        script {
          def dockerImage = docker.build(imageName) 
          withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
            dockerImage.push()
            sh 'docker rmi $imageName'
          }
        }
      }
    }
    stage('Deploy to EC2 instance') {
      when {
        branch 'develop'
      }
      environment {
        containerName = 'math-api'
      }
      steps {
        withCredentials(
          bindings: [sshUserPrivateKey(
            credentialsId: 'ec2-ssh-credentials', // ec2-ssh-credentials
            keyFileVariable: 'identityFile',
            passphraseVariable: 'passphrase',
            usernameVariable: 'user'
          )
        ]) {
          script {
            sh '''
              chmod +x ./scripts/deploy.sh
              ssh -o StrictHostKeyChecking=no -i $identityFile $user@$ec2Instance \
              APP_PORT=$appPort CONTAINER_NAME=$containerName IMAGE_NAME=$imageName bash < ./scripts/deploy.sh
            '''
          }
        }
      }
    }
  }
}