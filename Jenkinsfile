pipeline {
  agent any
  environment {
    DOCKER_IMAGE_NAME = "xledbd/train-schedule"
  }
  stages {
    stage('Build') {
      steps {
        echo 'Running build'
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    }
    stage('Build Docker Image') {
      when {
        branch 'master'
      }
      steps {
        script {
          app = docker.build(DOCKER_IMAGE_NAME)
          app.inside {
            sh 'echo $(curl localhost:7999)'
          }
        }
      }
    }
    stage('Push Docker Image') {
      when {
        branch 'master'
      }
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_login') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
          }
        }
      }
    }
    stage('DeployToProduction') {
      when {
        branch 'master'
      }
      steps {
        input 'Deploy to Production?'
        milestone(1)
        withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          script {
            sh "sshpass -p '$USERPASS' -v scp ./train-schedule-kube.yml $USERNAME@$control_ip:/tmp/"
            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$control_ip \"envsubst < /tmp/train-schedule-kube.yml | kubectl apply -f -\""
            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$control_ip \"rm /tmp/train-schedule-kube.yml\""
          }
        }
      }
    }
  }
}
