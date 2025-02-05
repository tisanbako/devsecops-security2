pipeline {
  agent any

  stages {
    stage ('Build Artifact') {
      sh "mvn clean package  -DskipTests=true"
      archive 'target/*jar'  //so the package can be downloaded
    }
  }
}