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
//DevSecOps
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
        withSonarQubeEnv('SonarQube') {   // withSonarQubeEnv is added during quality gate (SonarQube) is the name of the server
            sh 'mvn clean verify sonar:sonar \
               -Dsonar.projectKey=numeric-application \
               -Dsonar.host.url=http://34.207.113.142:9000 \
               -Dsonar.login=sqp_f5f516ce0ac5dfddfe359f2b8f8a5d2b490a0ea0'
        }
        timeout(time: 2, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true 
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







