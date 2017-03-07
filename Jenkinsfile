node ('host') {
    tool name: 'java8', type: 'jdk'
	tool name: 'gradle3.3', type: 'gradle'

prepStatus = "\nPreparation Stage: [\u274C]"
buildStatus = "\nBuilding code Stage: [\u274C]"
testStatus = "\nTesting Stage: [\u274C]"
packStatus = "\nPackaging and Publishing results Stage: [\u274C]"
askStatus = "\nAsking for manual approval Stage: [\u274C]"
deplStatus = "\nDeployment Stage: [\u274C]"
trigStatus = "\nTriggering job and fetching artefact Stage: [\u274C]"

withEnv(["JAVA_HOME=${tool 'java8'}","PATH+GRADLE=${tool 'gradle3.3'}/bin","PATH+JAVA=${tool 'java8'}/bin"]) {

try {

 stage('Preparation')
   {    
    	echo "##########Preparation##########"
    	checkout([$class: 'GitSCM', branches: [[name: 'yskrabkou']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/MNT-Lab/mntlab-pipeline']]])

    prepStatus = "\nPreparation Stage: [\u2705]"
   }

  stage('Building code') 
   {
   		echo "##########Building code##########"   		
   		sh 'gradle build'

   		buildStatus = "\nBuilding code Stage: [\u2705]"
   }

   stage('Testing') 
   {
   		echo "##########Testing##########"
   		parallel (
   		unit_tests: {sh 'gradle test'},
   		jacoco_tests: {sh 'gradle jacoco'},
   		cucumber_tests: {sh 'gradle cucumber'}
   		)

   		testStatus = "\nTesting Stage: [\u2705]"
   }

    stage('Triggering job and fetching artefact after finishing') 
   {
    	echo "##########Triggering job and fetching artefact##########"  
   	    //build job: 'MNTLAB-yskrabkou-child1-build-job', parameters: [[$class: 'GitParameterValue', name: 'BRANCH_NAME', value: 'yskrabkou'], string(name: 'WORKSPACE', value: "${WORKSPACE}")]
   	    //step ([$class: 'CopyArtifact', projectName: 'MNTLAB-yskrabkou-child1-build-job', filter: 'yskrabkou_dsl_script.tar.gz']);
        build job: "MNTLAB-${BRANCH_NAME}-child1-build-job", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: "${BRANCH_NAME}"]]
        step ([$class: 'CopyArtifact', projectName: "MNTLAB-${BRANCH_NAME}-child1-build-job"]);
   	    

   	    trigStatus = "\nTriggering job and fetching artefact Stage: [\u2705]"
   }

     stage('Packaging and Publishing results') 
   {
   		artefactName = sh (script: "basename ${WORKSPACE}" + '.jar', returnStdout: true) 
  		echo "ARTEFACT NAME: ${artefactName}"
  		sh 'echo $BRANCH_NAME'
   		echo "##########Packaging and Publishing results##########" 
   		sh "tar -zxf ${BRANCH_NAME}_dsl_script.tar.gz"
   		sh "cp build/libs/\$(basename \${WORKSPACE}).jar ."
   		//sh "cp build/libs/$artefactName ./"
   		sh "tar czvf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz \$(basename \${WORKSPACE}).jar jobs.groovy Jenkinsfile"
   		archiveArtifacts artifacts: 'pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz', excludes: null

   		packStatus = "\nPackaging and Publishing results Stage: [\u2705]"
   }

    stage('Asking for manual approval') 
    timeout(time:8, unit:'HOURS')
   {
   		echo "##########Asking for manual approval##########"
   	    input message:'Approve Deployment?'

        askStatus = "\nAsking for manual approval Stage: [\u2705]"
   }

   stage('Deployment') 
   {
    	echo "##########Deployment##########"
   		sh 'java -jar \$(basename \${WORKSPACE}).jar'

   	    deplStatus = "\nDeployment Stage: [\u2705]"
   }

}
catch (ex)
	{
   
	}

}
   stage('Sending status') 
   {
   		echo "###################Sending status###################"
   		echo prepStatus + buildStatus + testStatus + trigStatus + packStatus + askStatus + deplStatus
   }


  }