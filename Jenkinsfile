pipeline {
  agent any

  options { timestamps() }

  environment {
    IMAGE_NAME = "lesleynzali/tp4-jenkins-cicd"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker image') {
      steps {
        sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
        sh 'docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest'
      }
    }

    stage('Test') {
      steps {
        sh 'docker run --rm $IMAGE_NAME:$BUILD_NUMBER npm test'
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "dockerhub", usernameVariable: "DH_USER", passwordVariable: "DH_PASS")]) {
          sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
          sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
          sh 'docker push $IMAGE_NAME:latest'
          sh 'docker logout'
        }
      }
    }

    stage('Deploy staging') {
      steps {
        sh 'docker rm -f tp4-staging || true'
        sh 'docker run -d --name tp4-staging -p 3001:3000 $IMAGE_NAME:latest'
      }
    }
    stage('Force Fail') {
      steps {
        sh 'exit 1'
     }
   }
  }

post {
  success {
    withCredentials([string(credentialsId: 'SLACK_URL', variable: 'SLACK_URL')]) {
      sh """
        curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"✅ Build SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}"}' \
        "$SLACK_URL"
      """
    }
  }

  failure {
    withCredentials([string(credentialsId: 'SLACK_URL', variable: 'SLACK_URL')]) {
      sh """
        curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"❌ Build FAILED: ${JOB_NAME} #${BUILD_NUMBER}"}' \
        "$SLACK_URL"
      """
    }
  }
}
}
