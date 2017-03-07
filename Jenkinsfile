#
import hudson.AbortException

node('host')
{
    tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'
    def stageStatus = ''
    
    try{
	    stage ('Preparation (Checking out from git)')
		{
		    stageStatus = 'Preparation (Checking out from git)'
		    checkout scm
		}

	    withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin", "PATH+JAVA=${tool 'java8'}/bin","JAVA_HOME=${tool 'java8'}"]) {
		stage ('Building')
		{
		    stageStatus = 'Building'
		    sh '''gradle build'''
		}

		stage ('Testing')
		{
                    stageStatus = 'Testing'
		    parallel junit:
			{sh 'gradle test'},

			jacoco:
			{sh 'gradle jacoco'},

			cucumber:
			{sh 'gradle cucumber'}
		}

		stage ('Triggering')
		{
		    stage = 'Triggering'
		    build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${BRANCH_NAME}"]]
		    step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job"]);
		    sh "tar -zxf ${BRANCH_NAME}*.tar.gz"
		}

		stage ('Packaging')
		{
		    stageStatus = 'Packaging'
		    sh '''cp ${WORKSPACE}/build/libs/$(basename $WORKSPACE).jar ${WORKSPACE}'''
		    sh '''tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile $(basename $WORKSPACE).jar'''
		    archiveArtifacts "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"
		    sh '''rm pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz'''
		}

		stage('Asking for approval')
		{
		    timeout(time:1, unit:'DAYS')
		    {
			input 'All done. We are ready for deployment. Proceed?'
		    }
		}

		stage('Deployment')
		{
	            stageStatus = 'Deployment'
		    sh '''java -jar $(basename ${WORKSPACE}).jar'''
		}

		stage('Status') {
		    echo 'Successful Deployment!'
		}
	    }
    }
    catch (error)
	    {
                stage('Status') {
	           echo "Error on $stageStatus"
		   throw error
		}
	    } 
}

/*NonCPS
def getLog() {
    def log = currentBuild.rawBuild.getLog(100)
	log.each{String line ->
		if (line.contains('Aborted by')){
			return line}	   
	}
}*/
