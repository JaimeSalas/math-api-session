pipeline {
  agent any
  environment {
    imageName = 'jaimesalas/math-api:latest'
    ec2Instance = 'ec2-13-36-240-133.eu-west-3.compute.amazonaws.com'
    appPort = 80
  }
  
  stages {
    stage('Install dependencies') {
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
    stage('E2E Tests') {
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
        withCredentials(
          bindings: [sshUserPrivateKey(
            credentialsId: 'ec2-ssh-credentials', // ec2-ssh-credentials
            keyFileVariable: 'identityFile',
            passphraseVariable: 'passphrase',
            usernameVariable: 'user'
          )
        ]) {
          script {
            def dockerImage = docker.build(imageName)
            withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
              dockerImage.push()
              sh 'docker rmi $imageName'
            }
          }
        }
      }
    }
    stage('Deploy to server') {
      when {
        branch 'develop'
      }
      environment {
        containerName = 'my-api-app'
      }
      steps {
        withCredentials([
          sshUserPrivateKey(
            credentialsId: 'ec2-ssh-credentials',
            keyFileVariable: 'identityFile',
            passphraseVariable: 'passphrase',
            usernameVariable: 'user'
          )
        ]) {
          sh '''
            ssh -o StrictHostKeyChecking=no -i $identityFile $user@$ec2Instance \
            APP_PORT=$appPort CONTAINER_NAME=$containerName IMAGE_NAME=$imageName bash < ./scripts/deploy.sh
          '''
        }
      }
    }
  }
}
