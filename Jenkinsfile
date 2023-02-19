pipeline {
  agent any
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
          app = docker.build("xledbd/train-schedule")
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
        withCredentials([sshUserPrivateKey(credentialsId: '5921f83e-940b-4420-a64f-d82570db8d65', usernameVariable: 'USERNAME', keyFileVariable: 'USERPASS')]) {
          script {
            sh "sshpass -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull xledbd/train-schedule:${env.BUILD_NUMBER}\""
            try {
            sh "sshpass -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
            sh "sshpass -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
            } catch (err) {
              echo: 'caught error: $err'
            }
            sh "sshpass -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 80:3000 -d xledbd/train-schedule:${env.BUILD_NUMBER}\""
          }
        }
      }
    }
  }
}
