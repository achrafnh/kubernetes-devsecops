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
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco(execPattern: 'target/jacoco.exec')
        }
      }
    }



stage("Quality Gate") {
    steps {
        script {
            def status = null
            
            timeout(time: 1, unit: 'HOURS') {
                // Submit the analysis to SonarQube
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=test -Dsonar.host.url=http://testdeux.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_cd8b1a2f1dc0cd69ff552f8621931dea02448b4c"
                }
                
                // Wait for the SonarQube analysis and collect the task ID
                def qg = waitForQualityGate()
                def taskId = qg.taskId
                
                // Check the status of the SonarQube task
                while (status != 'SUCCESS' && status != 'FAILED' && status != 'CANCELED') {
                    sleep 10 // Wait for 10 seconds between each check
                    withCredentials([string(credentialsId: 'sonar-password', variable: 'SONAR_PASSWORD')]) {
                        status = sh(returnStatus: true, script: "curl -sS -u admin:$SONAR_PASSWORD 'http://testdeux.eastus.cloudapp.azure.com:9000/api/ce/task?id=${taskId}' | jq -r '.task.status'")
                    }
                }
            }
            
            // Handle the status accordingly
            if (status == 'SUCCESS') {
                echo "SonarQube analysis completed successfully."
            } else {
                error "SonarQube analysis failed or was canceled."
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
