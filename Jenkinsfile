node
{
// Configurate tools as they set into Jenkins  
    tool name: 'JDK', type: 'jdk'	
    tool name: 'gradle3.3', type: 'gradle'
    withEnv(["PATH+GRADLE=${tool 'gradle3.3'}/bin","JAVA_HOME=${tool 'JDK'}","PATH+JAVA=${tool 'JDK'}/bin"])
    {
// Choose repos               
	  stage ('Preparation (Checking out).')
	    { git url:'https://github.com/MNT-Lab/mntlab-pipeline.git', branch:'akaminski' }
	    
//Build with gradle
	  stage ('Building code.')
	    { sh 'gradle build'; }
	    
//Start parallel test with gradle            
	  stage ('Tests')
	    {
               parallel JUnit:
                { sh 'gradle test'; },
                Jacoco:
                { sh 'gradle cucumber';},
                Cucumber:
                { sh 'gradle jacoco';  }
            }
            
//Start another job with transfer var BRANCH_NAME , recieve artifact with plug-in CopyArtefact       
	  stage ('Triggering job and fetching artefact ')
	    {
              build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${BRANCH_NAME}"]]
              step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job",filter: '${BRANCH_NAME}_dsl_script.tar.gz']);
            }

            
	  stage ('Packaging and Publishing results.')
	      {
                sh '''
                cp ${WORKSPACE}/build/libs/$(basename "$PWD").jar ${WORKSPACE}/${BRANCH_NAME}-${BUILD_NUMBER}.jar
                tar -zxvf ${BRANCH_NAME}_dsl_script.tar.gz jobs.groovy
                tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile ${BRANCH_NAME}-${BUILD_NUMBER}.jar
                ''';
                archiveArtifacts artifacts: "pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz"
	      }
            
        stage ('Manual approval.')
	    {
		input 'deploy'
            /* timeout(time:3, unit:'MINUTES') 
                {
                    input message:'Approve deployment?'
                } */
            }
            

        stage ('Deployment.')
	    {
             sh 'java -jar ${BRANCH_NAME}-${BUILD_NUMBER}.jar'
            }
            

     }  
}
