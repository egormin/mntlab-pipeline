node ('host') {
   tool name: 'gradle3.3', type: 'gradle'
   tool name: 'java8', type: 'jdk'
   withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin","JAVA_HOME=${tool 'java8'}","PATH+JAVA=${tool 'java8'}/bin"])
   {
      try {
	stage('Checking out') {
		result = "error 1:nothing to check"
		git url: 'https://github.com/MNT-Lab/mntlab-pipeline.git', branch: 'amatveenko'
		}
    
	stage('Building code') {
		result = "error 2:could't build"
        	sh '''    
		java -version
        	gradle clean build
        	'''
        	}
    
	stage('Testing') {
		result = "error 3:tests didn't pass"
    		parallel junit: {
		sh 'gradle test'}, 
		jacoco: {
		sh 'gradle jacoco'}, 
		cucumber: {
		sh 'gradle cucumber'};
    		}
	
 
	stage ('Triggering job and fetching') {
		result = "error 4:couldn't pass the child job"
		build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "origin/${BRANCH_NAME}"]]
        	step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job"])
		}
	
	stage ('Packaging and Publishing results') {
		result = "error 5:couldn't pack"
		sh '''
		cp ${WORKSPACE}/build/libs/$(basename "$WORKSPACE").jar ${WORKSPACE}/gradle-simple.jar
		tar zxf ${BRANCH_NAME}_dsl_script.tar.gz jobs.groovy
		tar czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile gradle-simple.jar
		'''
        	archiveArtifacts artifacts: "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"
		}

	stage ('Asking for manual approval')
		timeout(time:5, unit:'MINUTES') {
		result = "error 6:Perhaps you didn't confirm the deployment"
		input message:'Approve deployment?'
		}
	
	stage('Deployment') {
		result = "error 7:couldn't deploy"
		sh ''' 
		java -version
		java -jar gradle-simple.jar
		ls -lh
		'''
		result = "SUCCESS"
		}
	
      }
      catch (Exception err) {
	currentBuild.result = 'FAILURE'
      }
	stage('Sending status') {
		echo "RESULT: ${currentBuild.result}, ${result}" 
	}
	
   }
}

// rm -rf *
