pipeline {
  agent any

  stages {
    
            stage('Build Artifact') {
                steps {
                  sh "mvn clean package -DskipTests=true"
                  archive 'target/*.jar' 
                }
            }
            stage('SonarQube analysis') {
              withSonarQubeEnv('My SonarQube Server') {
                sh 'mvn clean package -DskipTests=true sonar:sonar'
              } // submitted SonarQube taskId is automatically attached to the pipeline context
            }
          

          // No need to occupy a node
          stage("Quality Gate"){
            timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
              def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
              if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
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
