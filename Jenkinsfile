pipeline {
  agent any
  parameters {
    booleanParam(name: 'CANARY_DEPLOYMENT', defaultValue: false, description: 'Deploy Canary?')
  }
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
        containerName = 'math-api'
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
    stage('Canary Deploy') {
      when {
        branch 'production'
      }
      steps {
        script {
          if (params.CANARY_DEPLOYMENT) {
            versioningLatestAndPushImage(imageName, 'v2')
            cleanLocalImages(imageName, 'v2')
            
            sh 'echo connect to kubernetes and apply canary deployment...'
          } else {
            versioningLatestAndPushImage(imageName, 'v1')
            cleanLocalImages(imageName, 'v1')
            withKubeConfig([credentialsId: 'K8S-FILE', serverUrl: 'https://CF0249BAED2CBFAE8A24D34CE5E6CF7C.sk1.eu-west-3.eks.amazonaws.com']) {
              sh 'kubectl apply -f kube/app-deployment.yaml'
            }
          }
        }
      }
    }
  }
}

void versioningLatestAndPushImage(String imageName, String version) {
  withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
      echo "pull latest"
      sh "docker pull ${imageName}:latest"
      sh "echo tagging to ${version}"
      sh "docker tag ${imageName}:latest ${imageName}:${version}"
      sh "echo pushing ${imageName}:${version}"
      sh "docker push ${imageName}:${version}"
  }
} 

void cleanLocalImages(String imageName, String version) {
  echo 'removing local images'
  sh "docker rmi ${imageName}:latest"
  sh "docker rmi ${imageName}:${version}"
}
