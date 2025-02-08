pipeline {
  agent any

  stages {
    stage ('Build Artifact') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar' // for the package to be doanloadedd
      }
    }

    stage ('Unit Tests - JUnit & Jacoco') {
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

    stage ('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post{
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }

    stage ('Sonarqube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
            sh 'mvn clean verify sonar:sonar \
                -Dsonar.projectKey=numeric-application \
                -Dsonar.host.url=http://107.21.88.72:9000 \
                -Dsonar.login=sqp_1c2fa44df4e4e020e2a2e3a045b2a43107087e52'
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true 

          }      
        }
       
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: ""]){
          sh 'printenv'
          sh 'docker build -t tisanbako/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push tisanbako/numeric-app:""$GIT_COMMIT""'
        }
         
      }  
    }
    stage ('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']){
          sh "sed -i 's#replace#tisanbako/numeric-app:${GIT_COMMIT}#g' k8s-tisan.yaml"
          sh "kubectl apply -f k8s-tisan.yaml"
        }
      }
    }

  }
}