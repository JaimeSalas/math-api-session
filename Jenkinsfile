pipeline {
  agent any
  environment {
      imageName = 'jaimesalas/my-api-app:latest'
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
    // stage('Build') {
    //   agent {
    //     docker {
    //       image 'node:14-alpine'
    //       reuseNode true
    //     }
    //   }
    //   steps {
    //     sh 'npm run build'
    //   }
    // }
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
  }
}