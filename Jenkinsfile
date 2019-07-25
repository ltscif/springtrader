library 'LEAD'
pipeline {
  agent {
    label "lead-toolchain-skaffold"
  }
  stages {
    stage('Build') {
      steps {
        notifyPipelineStart()
        notifyStageStart()
        container('skaffold') {
          sh "skaffold build --file-output=image.json"
        }
      }
      post {
        success {
          notifyStageEnd()
        }
        failure {
          notifyStageEnd([result: "Build Failed"])
        }
      }
    }
    stage("Deploy to Staging") {
      when {
          branch 'master'
      }
      environment {
        TILLER_NAMESPACE = "${env.stagingNamespace}"
        ISTIO_DOMAIN   = "${env.stagingDomain}"
      }
      steps {
        notifyStageStart()
        container('skaffold') {
          sh "skaffold deploy -a image.json -n ${TILLER_NAMESPACE}"
        }
      }
      post {
        success {
          notifyStageEnd([status: "Successfully deployed to staging:\nspringtrader.${env.stagingDomain}/spring-nanotrader-web/"])
        }
        failure {
          notifyStageEnd([result: "Deploy failed"])
        }
      }
    }
    stage ('Manual Ready Check') {
      when {
        branch 'master'
      }
      options {
        timeout(time: 30, unit: 'MINUTES')
      }
      input {
        message 'Deploy to Production?'
      }
      steps {
        echo "Deploying"
      }
    }
    stage("Deploy to Production") {
      when {
          branch 'master'
      }
      environment {
        TILLER_NAMESPACE = "${env.productionNamespace}"
        ISTIO_DOMAIN   = "${env.productionDomain}"
      }
      steps {
        notifyStageStart()
        container('skaffold') {
          sh "skaffold deploy -a image.json -n ${TILLER_NAMESPACE}"
        }
      }
      post {
        success {
          notifyStageEnd([status: "Successfully deployed to production:\nspringtrader.${env.productionNamespace}/spring-nanotrader-web/"])
        }
        failure {
          notifyStageEnd([result: "fail"])
        }
      }
    }
  }
}

