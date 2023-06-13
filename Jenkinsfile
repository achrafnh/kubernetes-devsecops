pipeline {
  agent any

  stages {
      stage("build & SonarQube analysis") {
          node {
              withSonarQubeEnv('My SonarQube Server') {
                 sh 'mvn clean package -DskipTests=true sonar:sonar'
                  archive 'target/*.jar' 
              }
          }
      }

stage('SonarQube analysis') {
withSonarQubeEnv('My SonarQube Server') {
sh 'mvn clean package sonar:sonar'
} // submitted SonarQube taskId is automatically attached to the pipeline context
}

stage("Quality Gate"){
timeout(time: 1, unit: 'HOURS') {
    def qg = waitForQualityGate()
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
