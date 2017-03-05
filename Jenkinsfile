node ('host') {
tool name: 'gradle3.3', type: 'gradle'
tool name: 'java8', type: 'jdk'
currentBuild.result = 'SUCCESS'
def result = ""
	withEnv (["PATH+GRADLE=${tool 'gradle3.3'}/bin", "JAVA_HOME=${tool 'java8'}"]) { try {
	stage('Preparation (Checking out)') {
		try {
		   git url:'https://github.com/MNT-Lab/mntlab-pipeline.git', branch:'mburakouski'
		} catch (err) {
			result = "Fail with Checking" 
		}
		
	}
	stage ('Building code'){
		try {
			gradle build
		} catch (err) {
			result = "Fail with Building code"
		}
	}	
  	stage ('Testing'){
		try {
    		parallel JUnit: {
      			gradle test
    		}, Jacoco: {
      			gradle cucumber
    		}, Cucumber: {
      			gradle jacoco
		} 
		} catch (err) {
			result = "Fail with Testing"
		}
    	failFast: true|false  
  	}
  	stage ('Triggering job and fetching artefact after finishing'){
   		try {
			build job: 'MNTLAB-${env.BRANCH_NAME}-child1-build-job', parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${env.BRANCH_NAME}"]]
            step ([$class: 'CopyArtifact', projectName: 'MNTLAB-${env.BRANCH_NAME}-child1-build-job', filter: '*.tar.gz']);
		} catch (err) {
			result = "Fail with Triggering job and fetching artefact"
		}
	}
  	stage ('Packaging and Publishing results'){
		try{
			sh 'cp ${WORKSPACE}/build/libs/$(basename "${WORKSPACE}").jar ${WORKSPACE}'
			sh 'tar xvzf ${BRANCH_NAME}_dsl_script.tar.gz'	
			sh 'tar cvzf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz Jenkinsfile jobs.groovy *.jar'
			archive 'pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz'
		} catch (err) {
				result = "Fail with Packaging and Publishing results"
		}
  	}
  	stage ('Asking for manual approval'){
		try {
			input "Deployment?"
		} catch (err) {
			result = "Fail with approval"
		}
  	}    
  	stage ('Deployment'){
		try {
			sh 'java -jar $(basename "${WORKSPACE}").jar'
		} catch (err) {
			result = "Fail with Deployment"
		}	
  	}
}
catch (err) {
	currentBuild.result = 'FAILURE'
}
  	stage ('Sending status'){
     		echo "RESULT: ${currentBuild.result} - ${result}"
	}
}
}
