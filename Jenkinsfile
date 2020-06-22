
pipeline {
    agent any
    stages {


	    stage('Lint HTML') {
            steps {
                sh 'tidy -q -e index.html'
            }
        }

        stage('Create kubernetes cluster') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        eksctl create cluster \
                        --name capstonecluster \
                        --version 1.16 \
                        --nodegroup-name standard-workers \
                        --node-type t2.small \
                        --nodes 2 \
                        --nodes-min 1 \
                        --nodes-max 3 \
                        --node-ami auto \
                        --region us-west-2 \
                        --ssh-access \
                        --ssh-public-key devops.pub \
                        --managed
                    '''
                }
            }
        }



        stage('Upload to AWS') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-devops') {
		        sh 'echo "Hello World with AWS"'
                s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index.html', bucket:'udacity-cloud-jenkins')
                }
            }
        }
    }
}