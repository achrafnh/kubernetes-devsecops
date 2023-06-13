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
         stage('Docker build & push a') {
            steps {
              withCredentials([string(credentialsId: 'pass-dh-nm', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh 'sudo docker login -u nylv -p $DOCKER_HUB_PASSWORD'
                    sh "sudo printenv"
                    sh 'sudo docker build -t nylv/numeric-app:""$GIT_COMMIT"" .'
                    sh 'sudo docker push nylv/numeric-app:""$GIT_COMMIT"" ' 
              }
            }
         }
        stage('Kube8') {
          steps {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              
            }
          }
        }
    }
}
