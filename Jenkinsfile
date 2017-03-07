#!groovy

try {

	node ('host') { timestamps { 
			tool name: 'java8', type: 'jdk'
		    tool name: 'gradle3.3', type: 'gradle'
		//	def jdktool = tool 'java8'
		//  def gradletool = tool 'gradle3.3'
		    withEnv(["JAVA_HOME=${tool 'java8'}","PATH+GRADLE=${tool 'gradle3.3'}/bin","PATH+JAVA=${tool 'java8'}/bin"]) {
		       			// cheking env
			        	sh '''
			            	echo $PATH
			            	echo $JAVA_HOME           
			        	''';
			        	//sh './gradlew build --stacktrace --info --debug'

			        			        
			            stage('Preparation (Checking out)') {
			            	//git url: 'https://github.com/sunofsparda/mntlab-pipeline.git', branch: 'master'
			            	//git url: 'https://github.com/MNT-Lab/mntlab-pipeline.git', branch: 'acherlyonok'
			            	//checkout([$class: 'GitSCM', branches: [[name: '*/acherlyonok']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '4da30b2e-bd0e-445d-89bb-3aca0cd96599', url: 'https://github.com/MNT-Lab/mntlab-pipeline.git']]])
			                checkout scm
			            }

			            stage('Building code') {
			               	sh 'gradle build'
			            	//sh './gradlew build --stacktrace --info --debug'
			            }

			            stage('Testing') {   	
			            	parallel (
			            		'JUnit Tests': {sh 'gradle test'},
			            		'Jacoco Tests': {sh 'gradle jacoco'},
			            		'Cucumber Tests': {sh 'gradle cucumber'}
			            	)
			            }
			    

			            stage('Triggering job and fetching artefact after finishing') {
			                echo 'Building MNTLAB-acherlyonok-child1-build-job'
			                build job: 'MNTLAB-acherlyonok-child1-build-job', parameters: [[$class: 'GitParameterValue', name: 'BRANCH_NAME', value: 'acherlyonok'], string(name: 'WORKSPACE', value: "${WORKSPACE}")]
			                step ([$class: 'CopyArtifact', projectName: 'MNTLAB-acherlyonok-child1-build-job', filter: '${BRANCH_NAME}_dsl_script.tar.gz']);
			                sh 'tar -xzf acherlyonok_dsl_script.tar.gz jobs.groovy'

			            }


			            stage('Packaging and Publishing results') {
			                echo 'Creating new artifact'
			                sh '''
			                    cp ${WORKSPACE}/build/libs/$(basename "$PWD").jar ${WORKSPACE}/${BRANCH_NAME}-${BUILD_NUMBER}-gradle-simple.jar
			                    tar -czf pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile ${BRANCH_NAME}-${BUILD_NUMBER}-gradle-simple.jar
			                ''';
			                archiveArtifacts artifacts: 'pipeline-${BRANCH_NAME}-${BUILD_NUMBER}.tar.gz', onlyIfSuccessful: true
			            }


			            stage('Asking for manual approval') {
                            try {
                                input "Badaboom?"
                            }
                            catch (err) {
                                def user = err.getCauses()[0].getUser()
                                echo "WASTED by:\n ${user}"
                                throw err
                            }
			            	
			            }


			            stage('Deployment') {
			                echo 'Deploying'  
			                sh 'java -jar ${BRANCH_NAME}-${BUILD_NUMBER}-gradle-simple.jar'
			                
			            }


			            stage('Sending status') {
                            env.status = "Object completed"
                            echo "$status"
                        }


			}

		    	
	}	}
}

catch(err){
	echo "WASTED.."
}
