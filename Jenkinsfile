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
        sh "mvn clean verify sonar:sonar \
				  -Dsonar.projectKey=numeric-application \
				  -Dsonar.projectName='numeric-application' \
				  -Dsonar.host.url=http://devsecops.infocodesolutions.com:9000 \
				  -Dsonar.token=sqp_1ad57f37df71c1dfec7b5df1577ddaffccdccf19"
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
}