pipeline {
 agent none
 stages {
   stage('Upstream pipeline') {
    steps {
     script {
node() {
    paramAValue = "github.com"
    paramBValue = "main"
    build job: 'my_downstream_job', parameters: [[$class: 'StringParameterValue', name: 'GIT_URL', value: sh(returnStdout: true, script: 'git config remote.origin.url').trim()], [$class: 'StringParameterValue', name: 'BRANCH_NAME', value: BRANCH_NAME]]
       }
     }
   }
 }
}
}
