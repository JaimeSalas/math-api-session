pipeline {
  agent any
  stages {
    stage('Install dependecies') {
      agent {
        docker {
          image 'node:14-alpine'
        }
      }
      steps {
        sh 'npm ci'
      }
    }
    stage('Build') {
      agent {
        docker {
          image 'node:14-alpine'
        }
      }
      steps {
        sh 'npm run build'
      }
    }
  }
}