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
    stage('CanaryDeploy') {
      when {
        branch 'master'
      }
      environment {
        CANARY_REPLICAS = 1
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          script {
            sh "envsubst < ./train-schedule-kube-canary.yml > /tmp/train-schedule-kube-canary.yml && sshpass -p '$USERPASS' -v scp /tmp/train-schedule-kube-canary.yml $USERNAME@$control_ip:/tmp/ && rm /tmp/train-schedule-kube-canary.yml"
            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$control_ip \"kubectl apply -f /tmp/train-schedule-kube-canary.yml && rm /tmp/train-schedule-kube-canary.yml\""
          }
        }
      }
    }
    stage('DeployToProduction') {
      when {
        branch 'master'
      }
      environment {
        CANARY_REPLICAS = 0
      }
      steps {
        input 'Deploy to Production?'
        milestone(1)
        withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          script {
            sh "envsubst < ./train-schedule-kube-canary.yml > /tmp/train-schedule-kube-canary.yml && sshpass -p '$USERPASS' -v scp /tmp/train-schedule-kube-canary.yml $USERNAME@$control_ip:/tmp/ && rm /tmp/train-schedule-kube-canary.yml"
            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$control_ip \"kubectl apply -f /tmp/train-schedule-kube-canary.yml && rm /tmp/train-schedule-kube-canary.yml\""
            sh "envsubst < ./train-schedule-kube.yml > /tmp/train-schedule-kube.yml && sshpass -p '$USERPASS' -v scp /tmp/train-schedule-kube.yml $USERNAME@$control_ip:/tmp/ && rm /tmp/train-schedule-kube.yml"
            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$control_ip \"kubectl apply -f /tmp/train-schedule-kube.yml && rm /tmp/train-schedule-kube.yml\""
          }
        }
      }
    }
  }
}
