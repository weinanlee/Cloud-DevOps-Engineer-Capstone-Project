
pipeline {
    agent any
    stages {
	    stage('Lint HTML') {
            steps {
                sh 'tidy -q -e index.html'
            }
        }
        stage('Upload to AWS') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-static') {
		        sh 'echo "Hello World with AWS"'
                s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index.html', bucket:'udacity-cloud-jenkins')
                }
            }
        }
    }
}