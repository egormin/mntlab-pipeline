node ('host') {
	tool name: 'gradle3.3', type: 'gradle'
	tool name: 'java8', type: 'jdk'
	withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin","JAVA_HOME=${tool 'java8'}","PATH+JAVA=${tool 'java8'}/bin"])
	{
  // try {
	stage('Checking out') { 
		git url: 'https://github.com/MNT-Lab/mntlab-pipeline.git', branch: 'amatveenko'
		}
    
	stage('Building code') {
        	sh '''    
		java -version
        	gradle build
        	'''
        	}
    
	//stage('Testing') 
    	//	parallel junit: {
	//	sh 'gradle test'}, 
	//	jacoco: {
	//	sh 'gradle jacoco'}, 
	//	cucumber: {
	//	sh 'gradle cucumber'}
    	//	}
 
	stage ('Triggering job and fetching') {
		build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "origin/${BRANCH_NAME}"]]
        	step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job"])
		}
	
	stage ('Packaging and Publishing results') {
		sh '''
		cp ${WORKSPACE}/build/libs/$(basename "$WORKSPACE").jar ${WORKSPACE}/gradle-simple.jar
		tar zxf ${BRANCH_NAME}_dsl_script.tar.gz jobs.groovy
		tar czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile gradle-simple.jar
		'''
        	archiveArtifacts artifacts: "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"
		}

	stage ('Asking for manual approval')
		timeout(time:5, unit:'MINUTES') {
		input message:'Approve deployment?'
		}
	
	stage('Deployment') {
		sh ''' 
		java -version
		java -jar gradle-simple.jar
		ls -lh
		'''
		}
	
	stage('Sending status') {
		echo "SUCCESS"
		}
   }


// rm -rf *
