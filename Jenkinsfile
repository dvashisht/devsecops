pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar' // so that they can be downloaded later on.
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"        
      }      
    }

    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }      
    }

    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn clean verify sonar:sonar \
				  -Dsonar.projectKey=numeric-application \
				  -Dsonar.projectName='numeric-application' \
				  -Dsonar.host.url=http://devsecops.infocodesolutions.com:9000"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
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
          sh 'sudo docker build -t vashishtd/docker-images:""$GIT_COMMIT"" .'
          sh 'docker push vashishtd/docker-images:""$GIT_COMMIT""'
        }
      }
    }

    stage('K8S Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
        sh "sed -i 's#replace#vashishtd/docker-images:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
        sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }      
    }
  }

  post { 
        always { 
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
        // success {

        // }
        // failure {

        // }
  }
}