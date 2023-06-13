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


    
      stage('Sonarqube - SAST') {
            steps {
              sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.projectName='devsecops'-Dsonar.host.url=http://testdeux.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_320a08090648bb7e06dd756c9c5b6e7082258f79" 
            }
      
           } 

    
         stage('Mutation Tests - PIT Tests') {
            steps {
             # sh "mvn org.pitest:pitest-maven:mutationCoverage" 
            }
            post{
              always{
              pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            
              }
            }
           } 


        stage('Docker Build and Push ') {
            steps {
             
               withCredentials([string(credentialsId: 'docker-hub-password', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh 'sudo docker login -u hrefnhaila -p $DOCKER_HUB_PASSWORD'
              sh "sudo printenv"
              sh 'sudo docker build -t hrefnhaila/numeric-app:""$GIT_COMMIT"" .'
              sh 'sudo docker push hrefnhaila/numeric-app:""$GIT_COMMIT"" ' 
                }
             
          
                
            }
        } 


            stage('Kubernetes deployment - DEV ') {
            steps {
             
                  withKubeConfig([credentialsId: 'kubeconf']) {
             
                 
                    sh "sed -i 's#replace#hrefnhaila/numeric-app:$GIT_COMMIT#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
             
                }
             
          
                
            }
        } 

    
     }
}
