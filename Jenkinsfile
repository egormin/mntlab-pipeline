node ('host'){
tool name: 'gradle3.3', type: 'gradle'
tool name: 'java8', type: 'jdk'
currentBuild.result = 'SUCCESS'
def result = []
	
withEnv (["PATH+GRADLE=${tool 'gradle3.3'}/bin", "JAVA_HOME=${tool 'java8'}"]) { 
try {
	stage('Preparation (Checking out)') {
		result.push("Fail with Checking")
		git url:'https://github.com/MNT-Lab/mntlab-pipeline.git', branch:'hvysotski'
		}
	stage ('Building code'){
		result.push("Fail with Building code")
		sh 'gradle clean build'
	}	
  	stage ('Testing'){
		result.push("Fail with Testing")
		parallel JUnit: {
      			sh 'gradle test'
    		}, Jacoco: {
      			sh 'gradle cucumber'
    		}, Cucumber: {
      			sh 'gradle jacoco'
		} 
		failFast: true|false  
  	}
  	stage ('Triggering job and fetching artefact after finishing'){
		result.push("Fail with Triggering job and fetching artefact")
   		build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: "${BRANCH_NAME}", value: "${BRANCH_NAME}"]]
            	step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job", filter: '*.tar.gz']);
		}
  	stage ('Packaging and Publishing results'){
		result.push("Fail with Packaging and Publishing results")
		sh 'cp ${WORKSPACE}/build/libs/$(basename "${WORKSPACE}").jar ${WORKSPACE}'
		sh 'mv $(basename "${WORKSPACE}").jar gradle-simple.jar '
		sh "tar xvzf ${BRANCH_NAME}_dsl_script.tar.gz"	
		sh "tar cvzf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz Jenkinsfile jobs.groovy *.jar"
		archiveArtifacts "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"
		}
  	stage ('Asking for manual approval'){
		result.push("Fail with approval")
		input "Deployment?"
		}    
  	stage ('Deployment'){
		result.push("Fail with Deployment")
		sh 'java -jar \$(basename \${WORKSPACE}).jar'
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
