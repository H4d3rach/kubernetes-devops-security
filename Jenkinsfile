pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
      stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }

      /*stage('SonarQube analysis') {
        steps{
        withSonarQubeEnv(installationName:'sonarqube') { // You can override the credential to be used, If you have configured more than one global server connection, you can specify the corresponding SonarQube installation name configured in Jenkins
          sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar'
        }
        }
      }

      stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }*/
      stage('Docker Build') {
        steps {
            withDockerRegistry([credentialsId: 'docker-hub', url:'']){
            sh 'echo USER: $(whoami)'
    sh 'echo HOST: $(hostname)'
    sh 'which docker || echo "docker no encontrado"'
    sh 'docker --version || echo "docker no existe"'
    sh 'which kubectl || echo "kubectl no encontrado"'
            sh 'docker build -t h4d3rach/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push h4d3rach/numeric-app:""$GIT_COMMIT""'
          }
        }
      }

      stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#h4d3rach/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}