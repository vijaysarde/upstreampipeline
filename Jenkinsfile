pipeline {
 agent any
 stages {
   stage('Upstream pipeline') {
    steps {
     script {
       node() {

         checkout scm
         build job: 'kedar_downstream', propagate: true,
         parameters: [[$class: 'StringParameterValue', name: 'GIT_URL', value: sh(returnStdout: true, script: 'git config remote.origin.url').trim()],
		      [$class: 'StringParameterValue', name: 'BRANCH_NAME', value: env.BRANCH_NAME]
		 ]
       }
	     sh '''
	     echo "Printing files at ${WORKSPACE}"
	     ls ${WORKSPACE}
	     cd ${WORKSPACE}/upstreampipeline
	     ls
	     '''
     }
   }
 }
}
}
