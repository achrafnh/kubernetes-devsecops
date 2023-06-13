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
              
withSonarQubeEnv('SonarQube') {
sh "mvn clean verify sonar:sonar -Dsonar.projectKey=test -Dsonar.projectName='test' -Dsonar.host.url=http://testdeux.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_cd8b1a2f1dc0cd69ff552f8621931dea02448b4c"
}

              
      
           } }

    stage("Quality Gate") {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            script {
                def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }
    }
}



    
     
        //stage('Mutation Tests - PIT Tests') {
        //    steps {
        //     sh "mvn org.pitest:pitest-maven:mutationCoverage"
        //    }
        //    post{
        //      always{
       //       pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'

       //       }
        //    }
       //  }
    
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
              sh "sed -i 's#replace#nylv/numeric-app:$GIT_COMMIT#g' k8s_deployment_service.yaml"
              sh "kubectl apply -f k8s_deployment_service.yaml"
            }
          }
        }
    }
}
