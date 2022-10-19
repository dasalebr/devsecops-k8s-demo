pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'  //test
            }
        } 
      stage('Unit Tests') {
            steps {
              sh "mvn test"
            
            }
      }
       stage('SonarQube - SAST') {
            steps {
              sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo-wiz.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_33928ae877988f8959fe5b87b3672ae8540c54f2"
          } 
      }
      stage('Vulnerability Scan - Docker') {
        steps {
          sh "mvn dependency-check:check"
        
        }
      }    
      stage('Docker Build and Push') {
        steps {
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t dasalebr81/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push dasalebr81/numeric-app:""$GIT_COMMIT"" '
        }
      }        
    }
      stage('Kubernetes Deployment - DEV') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "sed -i 's#replace#dasalebr81/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
              }
            }
      }
     post { 
        always { 
             junit 'target/surefire-reports/*.xml'
             jacoco execPattern: 'target/jacoco.exec'
             dependencyCheckPublisher pattern: 'target/dependenct-check-report.xml'
        }
  }

 // success {

  //}
  //failure {

  //}
//}