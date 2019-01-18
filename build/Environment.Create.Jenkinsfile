#!groovy

pipeline {
  agent {
    label 'host'
  }
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '100', daysToKeepStr: '365'))
  }

  parameters {
    choice(name: 'NodeRedLocation', choices: ["local", "remote"], description: 'local or remote')
    string(name: 'RemoteFolder', defaultValue: 'c:/temp/oop/NodeRed or /tmp/oop/NodeRed', description: 'Remote location of Node-Red')
    string(name: 'HueBridgeIp', defaultValue: '192.168.200.20', description: 'The IP of the Hue Bridge')
    string(name: 'HueDimmerId', defaultValue: '', description: 'The id of the Hue Dimmer')
    string(name: 'HueLightId', defaultValue: '', description: 'The id of the Hue Lamp')
  }

  environment {
    BuildName = "Hue.${BUILD_NUMBER}"
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

    stage("Startup service local") {
      when{
        expression { params.NodeRedLocation == "local" }
      }
      steps {
        withCredentials([usernameColonPassword(credentialsId: "lidop", variable: "cred")]) {
          sh "ansible-playbook nodered/service.yml -vv --e \"state=present\""
        }
      }
    }

    stage("Startup service remote") {
      options { skipDefaultCheckout() }
      agent {
        label 'temp'
      }
      when {
        expression { params.NodeRedLocation == "remote" }
        beforeAgent true
      }
      steps {
          dir("${params.RemoteFolder}/node-red") {
            unstash "data"
          }

          script {
            if (isUnix()) {
              sh "echo #!/bin/bash\" >> ${params.RemoteFolder}/start.sh" 
              sh "echo ${params.RemoteFolder}/node/osx/bin/node ${params.RemoteFolder}/node-red/red.js >> ${params.RemoteFolder}/start.sh" 
      	      sh "chmod +x ${params.RemoteFolder}/start.sh"
              sh "open -a Terminal ${params.RemoteFolder}/start.sh"
            } else {
              dir("${params.RemoteFolder}") {
                bat "start ${params.RemoteFolder}/node/windows/node.exe ${params.RemoteFolder}/node-red/red.js"
              }
            }
          }

        }
      }

  }

    post { 
    always { 
      script {

        currentBuild.description = "goto <a href=${BASE_URL}/port/1880/>Node-Red</a>"

      }
    }

  }

}
