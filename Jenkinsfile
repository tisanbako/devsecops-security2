pipeline {
  agent any

  stages {
    stage ('Build Artifact') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar' // for the package to be doanloadedd
      }
    }

    stage ('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Build and Push Docker Image') {
      steps {
          script {
              def IMAGE_NAME = "tisanbako/numeric-app"
              def IMAGE_TAG = "$GIT_COMMIT"

              // Login to Docker Hub (Ensure credentials are stored in Jenkins)
             withCredentials([usernamePassword(credentialsId: "dockerhub")]) {
                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
             }

             // Build Docker Image
             sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."

             // Push Docker Image
             sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
          }
      }  
    }

  }
}