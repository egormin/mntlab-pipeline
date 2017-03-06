node ('host') {
    tool name: 'gradle3.3', type: 'gradle'
    tool name: 'java8', type: 'jdk'
    withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin","JAVA_HOME=${tool 'java8'}","PATH+JAVA=${tool 'java8'}/bin"])
    {

stage('Checking out') { 
    git url: 'https://github.com/MNT-Lab/mntlab-pipeline.git', branch: 'amatveenko'
    }
    
stage('Building code') {
        sh '''      
        gradle clean build
        '''
        }
        // chmod +x gradlew
    	// ./gradlew build
    	
    
stage('Testing') 
    parallel junit: {
	sh 'gradle test'}, 
	jacoco: {
	sh 'gradle jacoco'}, 
	cucumber: {
	sh 'gradle cucumber'}
    }
 
stage ('Triggering job and fetching') {
	build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${BRANCH_NAME}"]]
        step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job"])
	}
	
stage('Deploy') {
    echo 'Deploying....'
	sh 'ls -lh'
    }  
}
