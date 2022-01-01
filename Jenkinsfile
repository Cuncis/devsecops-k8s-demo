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
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://c79c-139-195-61-58.ngrok.io -Dsonar.login=2696a1696da7ddc6039af4c2fe0572d037cc278c"
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
