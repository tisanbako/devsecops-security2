pipeline {
  agent any
  environment {
    NVD_API = credentials('NVD_API_KEY')  // Reference API key securely
  }

  stages {
    //stage('Clear Dependency-Check Cache') {
      //steps {
        //sh 'rm -rf ~/.dependency-check}
    //}

    stage ('Build Artifact') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archiveArtifacts 'target/*.jar' // for the package to be doanloadedd
      }
    }

    stage ('Unit Tests - JUnit & Jacoco') {
      steps {
        sh "mvn test"
      }
    }
//DevSecOps
    stage ('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }

      post { 
        always { 
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }

    stage ('Sonarqube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {   //(SonarQube) is the name of the server configured on jenkins  withSonarQubeEnv is added during quality gate (SonarQube) is the name of the server
            sh 'mvn clean verify sonar:sonar \
               -Dsonar.projectKey=numeric-application'
               //-Dsonar.host.url=http://3.84.151.147:9000' 
               //-Dsonar.login=sqa_9186b9511e30542a4236574f385d630ea7604735'
        }
        timeout(time: 2, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true 
        }
       
      }  
    }

    stage('Dependency Check') {
            steps {
                sh '''
                    /opt/dependency-check/bin/dependency-check.sh \
                    --project MyApp \
                    --scan . \
                    --format HTML \
                    --out dependency-check-report \
                    --nvdApiKey $NVD_API
                '''
            }
        }

    //stage ('Dependency Check Scan') {
      //steps {
        //script {
          //sh '''
          //# Run OWASP Dependency-Check
          ///opt/dependency-check/bin/dependency-check.sh --project "MyApp" --scan . --format HTML --out dependency-check-report --nvdApiKey $NVD_API_KEY
          //'''
        //}
      //}
      //post {
        //always {
          //dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        //}
      //}
    //}

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
  post { 
        always { 
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml' 
          //archiveArtifacts artifacts: '**/dependency-check-report.html', fingerprint: true
        }
  }
}