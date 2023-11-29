pipeline {
  agent {
    label "jenkins-agent"
  }
  environment {
    APP_NAME = "complete-production-e2e-pipeline"
  }

  stages{
    stage("Cleanup Workspace"){
      steps{
        cleanWs()
      }
    }

    stage("Checkout from SCM"){
      steps{
        git branch: 'main', credentialsId: 'github', url: 'https://github.com/saqlaink/gitops-complete-production-e2e-pipeline'
      }
    }

    stage("Update the Deployment Tags"){
      steps {
        sh """
          cat deployment.yaml
          sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
          cat deployment.yaml
        """
      }
    }

    stage("Push the changed deployment file to git"){
      steps {
        sh """
          git config --global user.name "saqlaink"
          git config --global user.email "iamsaqlain25@gmail.com"
          git add deployment.yaml
          git commit -m "Updated deployment.yaml file"
        """
        withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
          sh "git push https://github.com/saqlaink/gitops-complete-production-e2e-pipeline main"
        }
      }
    }
  }
  post {
      always {
        script {
          def status = currentBuild.result ?: 'UNKNOWN'
          def color 
          switch(status) {
            case 'SUCCESS':
              color = 'good'
              break
            case 'FAILURE':
              color = 'danger'
              break
            default:
              color = 'warning'
          }
          slackSend(channel: "#jenkins", message: "Update Deployment ${status.toLowerCase()} for ${env.JOB_NAME} (${env.BUILD_NUMBER})\n More info at: ${env.BUILD_URL}", iconEmoji: ":jenkins:", color: color)
        }
      }
    }
}