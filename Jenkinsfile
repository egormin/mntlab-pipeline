#!groovy

//Setting parameters for current node: tools, PATH, HOME
//node('master') {
node('host') {	
	tool name: 'java8', type: 'jdk'
	tool name: 'gradle3.3', type: 'gradle'
	env.JAVA_HOME="${tool 'java8'}"
	env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
	env.GRADLE_HOME="${tool 'gradle3.3'}"
        env.PATH="${env.GRADLE_HOME}/bin:${env.PATH}"
//      withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin"])
//      withEnv(["JAVA_HOME=${tool 'java8'}"])


//ENbling errors cathing
	def err = null
	currentBuild.result = "SUCCESS"

	try {

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
} 

/*catch (InterruptedException e) {
        currentBuild.result = "ABORTED"
	sh "echo ${e}"
        mail body: "project build error: ${e}" ,
        subject: 'project build failed',
        to: 'n.g.kuznetsov@gmail.com'
    	throw e
    } 
*/

catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e){

        echo "the job was cancelled or aborted"
        sh "echo ${e}"
         }
	throw e
catch (e) {
        currentBuild.result = "FAILED"
        sh "echo ${e}"
        mail body: "project build error: ${e}" ,
        subject: 'project build failed',
        to: 'n.g.kuznetsov@gmail.com'
        throw e
    }



/*catch (caughtError) {
    err = caughtError
    currentBuild.result = "FAILURE"

	mail body: "project build error: ${err}" ,
	subject: 'project build failed',
	to: 'n.g.kuznetsov@gmail.com'
} */

finally {

    /* Must re-throw exception to propagate error */
    if (err) {
        throw err
	}	
    }
}
