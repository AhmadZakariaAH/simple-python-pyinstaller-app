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
	
	archiveArtifacts 'dist/add2vals-image'

        echo 'Pipeline has finished succesfully.'
        sleep time:1, unit: 'MINUTES'
    }
}
