node('host'){
    tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'
  
    prepStatus = "\nPreparation: [\u26c8]"
    buildStatus = "\nBuilding code: [\u26c8]"
    testStatus = "\nTesting: [\u26c8]"
    trigStatus = "\nTriggering job and fetching artefact Stage: [\u26c8]"
    packStatus = "\nPackaging and Publishing results: [\u26c8]"
    aprStatus = "\nAsking for manual approval: [\u26c8]"
    deployStatus = "\nDeployment: [\u26c8]"
    

withEnv(["JAVA_HOME=${ tool 'java8' }", "PATH+GRADLE=${tool 'gradle3.3'}/bin", "PATH+JAVA=${tool 'java8'}/bin"]) { 

try{ 
stage ('Preparation')
    {
    git branch: 'akutsko', url: 'https://github.com/MNT-Lab/mntlab-pipeline.git'
    prepStatus = "\nPreparation: [\u2600]"
    }

stage ('Building code')
    {
    sh 'gradle build'
    buildStatus = "\nBuilding code: [\u2600]"
    }
stage ('Testing code'){
    parallel Unit_Test: {sh 'gradle test'
             }, Jacoco_Test: {sh 'gradle jacoco'
             }, cucumber_Test: {sh 'gradle cucumber'
             }
    testStatus = "\nTesting: [\u2600]"
    }
stage ('Triggering job and fetching artefact after finishing'){
    build job: 'MNTLAB-akutsko-child1-build-job', parameters: [string(name: 'BRANCH_NAME', value: 'akutsko')], quietPeriod: 0
    step ([$class: 'CopyArtifact',
          projectName: 'MNTLAB-akutsko-child1-build-job',
          filter: 'akutsko_dsl_script.tar.gz']);
    trigStatus = "\nTriggering job and fetching artefact Stage: [\u2600]"
    }
stage ('Packaging and Publishing results'){
    sh 'cp build/libs/$(basename "$PWD").jar akutsko-v1.${BUILD_NUMBER}.jar'
    sh 'tar -xf akutsko_dsl_script.tar.gz jobs.groovy'
    sh 'tar -cvzf pipeline-akutsko-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile  akutsko-v1.${BUILD_NUMBER}.jar'
    archiveArtifacts artifacts: 'pipeline-akutsko-${BUILD_NUMBER}.tar.gz', excludes: null
    packStatus = "\nPackaging and Publishing results: [\u2600]"
    }
stage ('Asking for manual approval'){
    input message: 'Do you want to deploy this artefact?', ok: 'Deploy'
    aprStatus = "\nAsking for manual approval: [\u2600]"
    } 
stage ('Deployment'){
    sh 'tar -xf pipeline-akutsko-${BUILD_NUMBER}.tar.gz akutsko-v1.${BUILD_NUMBER}.jar'
    sh 'java -jar akutsko-v1.${BUILD_NUMBER}.jar'
    deployStatus = "\nDeployment: [\u2600]"
    }
  }
catch (ERROR){}
    stage('Sending status') 
   {
	echo prepStatus + buildStatus + testStatus + trigStatus + packStatus + aprStatus + deployStatus
       """ if ( prepStatus!="2600" || buildStatus!="2600" || testStatus!="2600" || trigStatus!="2600" || packStatus!="2600" || aprStatus!="2600" || deployStatus!="2600" ){currentBuild.result = 'FAILURE'}
        else {currentBuild.result = 'SACCESS'}"""
   }

}
}
