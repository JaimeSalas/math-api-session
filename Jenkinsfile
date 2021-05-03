pipeline {
  agent any
  environment {
    imageName = 'jaimesalas/math-api:latest'
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
        ec2Instance = 'ec2-3-8-33-70.eu-west-2.compute.amazonaws.com'
        appPort = 80
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