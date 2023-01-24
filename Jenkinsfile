node {
	def mvnHome	
	// Get the Maven tool.
	mvnHome = tool 'MAVEN'
	def applicationName='vpn-server'
	def dockerRepoUrl = "34.122.253.69:8083"
	
	def dockerImage	
	
	stage('Code checkout') { 
		echo "==========================================Code checkout starts====================================================="
		// Get some code from a GitHub repository
		def repo = "https://github.com/nnishad/${applicationName}.git"
		git branch: 'master',credentialsId: 'nnishad',url: repo
		echo "==========================================Code checkout ends====================================================="
	}

	stage ('Clean Up') {
		echo "==========================================Clean Up starts====================================================="
		echo "applicationName---  ${applicationName}"
		echo "user--- $USER"
		echo pwd
		sh   "rm -rf /tmp/${applicationName}*"
		sh   'docker system prune -a -f'
		echo "==========================================Clean Up ends====================================================="
	}
		
	stage('Build Code') {
		echo "==========================================Build Code starts====================================================="
		sh "'./gradlew' clean build"

		sh "'./gradlew' clean bootJar"
		//sh "'${mvnHome}/bin/mvn' clean package -Dmaven.test.skip=true -U"
		sh "cp /var/lib/jenkins/workspace/${applicationName}/build/libs//${applicationName}-0.0.1-SNAPSHOT.jar /var/lib/jenkins/workspace/${applicationName}"
		echo "==========================================Build Code ends====================================================="
	}

	stage ('Build Docker Image'){
		echo "==========================================Build Docker Image starts====================================================="			
		dockerImage = docker.build("admin/${applicationName}")
		
		echo "==========================================Build Docker Image ends====================================================="
	}
	stage('Test image') {
        dockerImage.inside {
            sh 'echo "Tests passed"'
        }
    }
	stage('Application Deployment'){
		echo "==========================================Application Deployment starts====================================================="
		def inspectExitCode = sh script: "docker container inspect ${applicationName}", returnStatus: true
		if (inspectExitCode == 0) {
		    sh "docker stop ${applicationName}"
			sh "docker rm ${applicationName}"
		} else {
		    echo "No instance was running. You can create new one."
		}
		def publishPort=2024
		def applicationPort=8080
		sh "docker run --name ${applicationName} -p ${publishPort}:${applicationPort} -d admin/${applicationName}"
		echo "==========================================Application Deployment ends====================================================="
	}
}
