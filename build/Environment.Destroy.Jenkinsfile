#!groovy

pipeline {
  agent {
    label 'host'
  }
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '100', daysToKeepStr: '365'))
  }

  environment {
    BuildName = "Destroy_Environment.${BUILD_NUMBER}"
  }

  stages {
    stage('Build Number') {
      steps {
        script {
          currentBuild.displayName = "Environment-Setup${BuildName}"
        }
      }
    }  

    stage("Configure service") {
      steps {
        withCredentials([usernameColonPassword(credentialsId: "lidop", variable: "cred")]) {
          sh "ansible-playbook nodered/config.yml -vv --e \"jenkins_url=${env.JENKINS_URL} project_name=${env.JOB_NAME} credential=${cred} state=present bridgeIp=${params.HueBridgeIp} lightId=${params.HueLightId} dimmerId=${params.HueDimmerId}\""
          dir("/var/lidop/hue/nodered/") {
            stash includes: 'flows.json, flows_cred.json', name: 'data'
          }
        }
      }
    }

    stage("Stop Service") {
      steps {
        sh "ansible-playbook nodered/service.yml -vv --e \"state=absent\""
      }
    }

  }
}
