#!Jenkinsfile

node {
	tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'

    stage('Checking out') {
    	git url: 'https://github.com/sunofsparda/mntlab-pipeline.git', branch: 'master'
    }
    stage('Building code’') {
    	sh 'chmod +x gradlew'
    	//gradle build
    }
    stage('Unit Tests') {
    	sh './gradlew test'
    }
    stage('Jacoco Tests') {
    	sh './gradle jacoco'
    }
    stage('Cucumber Tests') {
    	sh './gradle cucumber'
    }
    stage('Deploy') {
    	echo 'Deploying....'
    }
    
}
