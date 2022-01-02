pipeline {
  agent any

  stages {
      stage('Build Artifact') {
        steps {
          sh "mvn clean package -DskipTests=true"
          archive 'target/*.jar' 
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

      stage('SonarQube - SAST') {
        steps {
          withSonarQubeEnv('SonarQube') {
            sh "mvn sonar:sonar \
                -Dsonar.projectKey=numeric-application \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.login=3fae0c6184264d1ddc81c7e150987302c5ed797f"
          }
          timeout(time:2, unit: 'MINUTES') {
            script {
              waitForQualityGate abortPipeline: true
            }
          }
        }
      }

      stage('Docker Build and Push') {
        steps {
          // sh "docker --version"
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'docker build -t cuncis1st/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push cuncis1st/numeric-app:""$GIT_COMMIT""'
          }
        }
      }

      stage('Kubernetes Deployment - DEV') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
      }
  }
}