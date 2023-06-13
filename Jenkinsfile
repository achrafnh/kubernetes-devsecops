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


        stage('Docker Build and Push ') {
            steps {
              withDockerRegistry([credentialsId: 'dockerhub', url: '$DOCKER_REGISTRY_URL']){
                
              
              sh "sudo printenv"
              sh 'sudo docker build -t hrefnhaila/numeric-app:""$GIT_COMMIT"" .'
              sh 'sudo docker push hrefnhaila/numeric-app:""$GIT_COMMIT"" ' 
                }
            }
        } 

    
     }
}
