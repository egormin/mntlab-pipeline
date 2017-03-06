#!groovy

//Setting parameters for current node: tools, PATH, HOME
node('master') {
//node('host') {	
	tool name: 'java8', type: 'jdk'
	tool name: 'gradle3.3', type: 'gradle'
	env.JAVA_HOME="${tool 'java8'}"
	env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
	env.GRADLE_HOME="${tool 'gradle3.3'}"
        env.PATH="${env.GRADLE_HOME}/bin:${env.PATH}"
//      withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin"])
//      withEnv(["JAVA_HOME=${tool 'java8'}"])


try {
	currentBuild.result = "SUCCESS"


//All stages step-by-step
stage '\u2780 Preparation (Checking out)'
	git branch: 'mkuzniatsou', url: 'https://github.com/MNT-Lab/mntlab-pipeline.git'
	sh 'env | sort'
	sh 'gradle -version'
        sh 'java -version'

stage '\u2781 Building code'
	sh 'gradle clean build'
  

stage '\u2782 Testing code'
	parallel JUnit: {
	sh 'gradle test'}, 
	Jacoco: {
	sh 'gradle jacoco'}, 
	Cucumber: {
	sh 'gradle cucumber'
	}

stage '\u2783 Triggering job and fetching artefact after finishing'
	build job: "MNTLAB-$BRANCH_NAME-child1-build-job", parameters: [string(name: 'BRANCH_NAME', value: "${BRANCH_NAME}")]
//	archiveArtifacts '{BRANCH_NAME}_dsl_script.tar.gz,jobs.groovy,script.sh'
	step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job"]);

stage '\u2784 Packaging and Publishing results'
       	sh '''
	cp ${WORKSPACE}/build/libs/$(basename $WORKSPACE).jar ${WORKSPACE}/${BRANCH_NAME}-${BUILD_NUMBER}.jar
	tar -zxvf ${BRANCH_NAME}_dsl_script.tar.gz jobs.groovy       	
	tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile ${BRANCH_NAME}-${BUILD_NUMBER}.jar'''
	archiveArtifacts "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"

stage '\u2785 Asking for manual approval'
	timeout(time:1, unit:'HOURS') {
	input 'Previous stage successful. Artefact is ready. Deploy this artefact?'
	}

stage '\u2786 Deployment'
	sh 'java -jar ${BRANCH_NAME}-${BUILD_NUMBER}.jar'

stage '\u2787 Sending status'
	echo "\u2705 RESULT: ${currentBuild.result}"
} //try end 


catch (hudson.AbortException e) {
        currentBuild.result = "ABORTED"
        echo "the job was cancelled or aborted"
	sh "echo ${e}"
	sh '''
	curl -u admin:admin ${BUILD_URL}/consoleText > console.txt
	ABORT_USER=$(grep \'^Aborted by.*$\' console.txt)
	echo $ABORT_USER
	'''
        mail body: "project build error: ${e}" ,
        subject: 'project build failed',
        to: 'n.g.kuznetsov@gmail.com'
	throw e
	} 

catch (error) {
        currentBuild.result = "FAILED"
        sh "echo ${error}"
        mail body: "project build error: ${error}" ,
        subject: 'project build failed',
        to: 'n.g.kuznetsov@gmail.com'
        throw error
	}
}
