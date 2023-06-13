pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }
        stage('Unit Tests') {
            steps {
              sh "mvn test"
            }
            post{
              always{
                junit 'target/surefire-reports/*.xml'
                jacoco(execPattern: 'target/jacoco.exec')
                }
            }
         }
          stage('Docker build & push') {
            steps {
              sh "sudo printenv"
              sh 'sudo docker build -t nylv/numeric-app:""$GIT_COMMIT"" .'
            }
         }
    }
}
