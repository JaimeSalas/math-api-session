pipeline {
    agent any
    environment {
      imageName = 'jaimesalas/math-api-app:latest'
      ec2Instance = 'ec2-3-8-33-70.eu-west-2.compute.amazonaws.com'
      appPort = 80
      githubAccount = "jaimesalas"
      githubRepoName = "math-api-session"
    }
    stages {
     stage('Notify GitHub build in progress') {
       steps {
         githubNotify(
           status: "PENDING",
           credentialsId: "github-commit-status-credentials",
           account: githubAccount,
           repo: githubRepoName,
           description: "Some checks haven't completed yet"
         )
       }
     }
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
          script {
            def dockerImage = docker.build(imageName)
            withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
              dockerImage.push()
              sh 'docker rmi $imageName'
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
   post {
     success {
       githubNotify(
         status: "SUCCESS",
         credentialsId: "github-commit-status-credentials",
         account: githubAccount,
         repo: githubRepoName,
         description: "All checks have passed"
       )
     }
     failure {
       githubNotify(
         status: "FAILURE",
         credentialsId: "github-commit-status-credentials",
         account: githubAccount,
         repo: githubRepoName,
         description: "Some checks were not successful"
       )
     }
   }
  }