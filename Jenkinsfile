node {
    
    stage('Build') {
        docker.image('python:latest').inside() {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside() {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }

        junit 'test-reports/results.xml'
    }

    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?'
    }

    stage('Deploy') {
        docker.image('python:latest').inside('-u root') {
            sh 'pip install pyinstaller'
            sh 'pyinstaller --onefile sources/add2vals.py'
        }
        
        archiveArtifacts 'dist/add2vals'
        
	// Create a Dockerfile

	sh '''
	echo "FROM ubuntu" > Dockerfile
	echo "WORKDIR /usr/local/bin" >> Dockerfile
	echo "COPY dist/add2vals /usr/local/bin/add2vals" >> Dockerfile
	echo "RUN chmod +x /usr/local/bin/add2vals" >> Dockerfile
	echo "ENTRYPOINT [\"/usr/local/bin/add2vals\"]" >> Dockerfile
	'''

	// Build add2vals-image

	sh "docker build -t add2vals-image:latest ."

	// Push image into docker registry

	withCredentials([usernamePassword(credentialsId: 'docker-credentials',
		usernameVariable: 'DOCKER_USERNAME',
		passwordVariable: 'DOCKER_PASSWORD')]) {
			def DOCKER_REPO = 'add2vals-repo'
			def DOCKER_TAG = 'latest'
        
			echo "docker-repo: ${DOCKER_REPO}"
			echo "docker-tag: ${DOCKER_TAG}" 
			sh '''
			echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
			'''
			// sh 'docker tag add2vals-image:latest $DOCKER_USERNAME/add2vals-repo:latest'
			sh "docker push ${env.DOCKER_USERNAME}/${env.DOCKER_REPO}:${env.DOCKER_TAG}"
	}

        echo 'Pipeline has finished succesfully.'
        sleep time:1, unit: 'MINUTES'
    }
}
