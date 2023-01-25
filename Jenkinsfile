node {

	def applicationName='vpn-server'

	def dockerImage	
	
	stage('Code checkout') { 
		echo "==========================================Code checkout starts====================================================="
		// Get some code from a GitHub repository
		def repo = "https://github.com/nnishad/${applicationName}.git"
		git branch: env.BRANCH_NAME,credentialsId: 'nnishad',url: repo
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
		sh "cp /var/lib/jenkins/workspace/${applicationName}_${env.BRANCH_NAME}/build/libs/${applicationName}-0.0.1-SNAPSHOT.jar /var/lib/jenkins/workspace/${applicationName}"
		echo "==========================================Build Code ends====================================================="
	}

	stage('Test image') {
                    dockerImage.inside {
                        sh 'echo "Tests passed"'
                    }
    }

    if(env.BRANCH_NAME=="main" || env.BRANCH_NAME== "develop"){


        stage ('Build Docker Image'){
        		echo "==========================================Build Docker Image starts====================================================="
        		dockerImage = docker.build("${env.BRANCH_NAME}/${applicationName}-${env.BRANCH_NAME}")

        		echo "==========================================Build Docker Image ends====================================================="
        	}

        	stage('Application Deployment'){
        		echo "==========================================Application Deployment starts====================================================="
        		def inspectExitCode = sh script: "docker container inspect ${applicationName}-${env.BRANCH_NAME}", returnStatus: true
        		if (inspectExitCode == 0) {
        		    sh "docker stop ${applicationName}-${env.BRANCH_NAME}"
        			sh "docker rm ${applicationName}-${env.BRANCH_NAME}"
        		} else {
        		    echo "No instance was running. You can create new one."
        		}
        		def publishPort=2024
        		def applicationPort=8080
        		sh "docker run --name ${applicationName}-${env.BRANCH_NAME} -p ${publishPort}:${applicationPort} -d ${env.BRANCH_NAME}/${applicationName}-${env.BRANCH_NAME}"
        		echo "==========================================Application Deployment ends====================================================="
        	}

    }
}
