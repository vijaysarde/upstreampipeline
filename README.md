## This is a jenkinsfile from upstream job which will trigger downstream job with the parameters that we are configuring in this jenkinsfile

Read more at https://docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-masters/how-to-pass-parameter-to-downstream-job-in-pipeline-job

```
pipeline {
 agent none
 stages {
   stage('Upstream pipeline') {
   steps {
     script {
       node() {
         checkout scm
         build job: my_downstream_job, propagate: true;
         parameters: [[$class: 'StringParameterValue', name: 'GIT_URL', value: sh(returnStdout: true, script: 'git config remote.origin.url').trim()]]
       }
     }
   }
 }
}
}
```

# This one is tried and tested
```
pipeline {
 agent none
 stages {
   stage('Upstream pipeline') {
    steps {
     script {
       node() {
         checkout scm
         build job: 'my_downstream_job', propagate: true,
         parameters: [[$class: 'StringParameterValue', name: 'GIT_URL', value: sh(returnStdout: true, script: 'git config remote.origin.url').trim()],
		      [$class: 'StringParameterValue', name: 'BRANCH_NAME', value: env.BRANCH_NAME]
		 ]
       }
     }
   }
 }
}
}
```

The code you provided is a Jenkins pipeline script that defines a Jenkins pipeline with a single stage called "Upstream pipeline". Let's break down the code to understand its components:



- `Pipeline`: Specifies the start of a Jenkins pipeline block.
- `agent none`: Indicates that this pipeline does not require a specific agent or executor. It means that the pipeline will not run on any specific Jenkins agent but will be executed on the master node.
- `stages`: Defines the stages of the pipeline.
- `stage('Upstream pipeline')`: Defines a single stage named "Upstream pipeline".
- `script`: Contains the script block that defines the actions to be performed within the stage.
- `node()`: Allocates a Jenkins agent to execute the following steps. In this case, it will be executed on the master node.
- `checkout scm`: Checks out the source code from the configured source code management (SCM) system for the current pipeline.
- `build job: my_downstream_job, propagate: true;`: Triggers a build of the "my_downstream_job" Jenkins job and allows the build to propagate to downstream jobs if successful.
- `parameters: [[$class: 'StringParameterValue', name: 'GIT_URL', value: sh(returnStdout: true, script: 'git config remote.origin.url').trim()]]`: Specifies a parameter named "GIT_URL" with the value obtained by executing the Git command `git config remote.origin.url`. This parameter can be used to pass information to downstream jobs.

In summary, this pipeline script defines a single stage called "Upstream pipeline". It checks out the source code, triggers the execution of a downstream job named "my_downstream_job", and provides the Git URL as a parameter to the downstream job.


# Here is another one example

Upstream Pipeline Jenkinsfile

```
pipeline {
   agent any

   stages {
       stage('Trigger Downstream Pipeline') {
           steps {
               script {
                   // Define the parameters to pass to the downstream pipeline
                   def skipSonar = true  // Set the value based on your logic
				   def appType = 'maven' // Set the appType based on your logic

                   // Trigger the downstream pipeline with parameters
                   build job: 'downstream-pipeline', parameters: [
                       booleanParam(name: 'SKIP_SONAR', value: skipSonar),
					   stringParam(name: 'APP_TYPE', value: appType)
                   ]
               }
           }
       }
   }
}
```

Downstream Pipeline Jenkinsfile

```
pipeline {
   agent any

   parameters {
       booleanParam(name: 'SKIP_SONAR', defaultValue: false, description: 'Skip Sonar Analysis')
   }

   stages {
       stage('Code Checkout') {
           steps {
               // Checkout source code from version control system (e.g., Git)
               // Example: git 'https://github.com/your/repo.git'
           }
       }

       stage('Sonar Analysis') {
           when {
               expression { !params.SKIP_SONAR }
           }
           steps {
               // Perform SonarQube analysis
               // Example: sh 'sonar-scanner'
           }
       }

       stage('Build App') {
           steps {
               // Determine the app type based on logic (e.g., Maven or Gradle)
               script {
                   def appType = params.APP_TYPE  

                   if (appType == 'maven') {
                       // Build the Maven application
                       sh 'mvn clean install'
                   } else if (appType == 'gradle') {
                       // Build the Gradle application
                       sh './gradlew build'
                   } else {
                       error "Unsupported app type: ${appType}"
                   }
               }
           }
       }

       stage('Upload Artifact to Repository') {
           steps {
               // Upload the built artifact to your artifact repository
               // Example: sh 'mvn deploy'
           }
       }

       stage('Build Docker Image and Push') {
           steps {
               // Build and push the Docker image
               // Example: sh 'docker build -t your-image . && docker push your-image'
           }
       }

       stage('Deploy App') {
           steps {
               // Deploy the application to the desired environment
               // Example: sh 'kubectl apply -f your-deployment.yaml'
           }
       }
   }
}
```
