node('host')
{
// Configurate tools as they set into Jenkins  
    tool name: 'java8', type: 'jdk'	
    tool name: 'gradle3.3', type: 'gradle'
    withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin","JAVA_HOME=${tool 'java8'}","PATH+JAVA=${tool 'java8'}/bin"])
    {
      try{
    
// Choose repos               
	  stage ('Preparation (Checking out)')
	    { git url:'https://github.com/MNT-Lab/mntlab-pipeline.git', branch:'akaminski' }
	    
//Build with gradle
	  stage ('Building code.')
	    { sh 'gradle build'; }
	    
//Start parallel test with gradle            
	  stage ('Tests')
	    {   parallel JUnit:
                { sh 'gradle test'},
                Jacoco:
                { sh 'gradle cucumber'},
                Cucumber:
                { sh 'gradle jacoco'}
            }
            
//Start another job with transfer var BRANCH_NAME , recieve artifact with plug-in CopyArtefact       
	  stage ('Triggering job and fetching artefact ')
	    {
              build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${BRANCH_NAME}"]]
              step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job",filter: '${BRANCH_NAME}_dsl_script.tar.gz']);
            }

            
	  stage ('Packaging and Publishing results ')
	      {
                sh '''
                cp ${WORKSPACE}/build/libs/$(basename "$PWD").jar ${WORKSPACE}/${BRANCH_NAME}-${BUILD_NUMBER}.jar
                tar -zxf ${BRANCH_NAME}_dsl_script.tar.gz jobs.groovy
                tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile ${BRANCH_NAME}-${BUILD_NUMBER}.jar
                ''';
                archiveArtifacts artifacts: "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"
	      }
//waiting for approval            
	  stage ('Manual approval ')
		{
		try {
		     input 'deploy'
		     } 
		catch (err) {
		    def user = err.getCauses()[0].getUser() //need toget rights 
		    echo "Aborted by:\n ${user}"
		    throw err
			    }
		}
//execute .jar with java
	  stage ('Deployment.')
	      { sh 'java -jar ${BRANCH_NAME}-${BUILD_NUMBER}.jar' }
	      
//if build successful show status             
	  stage ('View status')
	      {
		env.status = " === Build Successful === "
		echo "$status"
	      }
	} //end try

	catch(err){
		env.status = " === Build FAILED  === "
		throw err
	  
	} // end catch
     } //end withEnv 
}
