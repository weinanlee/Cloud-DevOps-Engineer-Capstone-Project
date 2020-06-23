pipeline {
    agent any
    stages {

        stage('Lint HTML') {
            steps {
                sh 'tidy -q -e *.html'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                    sh '''
                        docker build -t weinanli/capstone-devops .
                    '''
                }
            }
        }

        stage('Push Image To Dockerhub') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                    sh '''
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker push weinanli/capstone-devops
                    '''
                }
            }
        }

        stage('Create conf file cluster') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        aws eks --region us-west-2 update-kubeconfig --name capstone-devops-cluster
                    '''
                }
            }
        }

        stage('Set kubectl context') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        kubectl config use-context arn:aws:eks:us-west-2:918317714877:cluster/capstone-devops-cluster
                    '''
                }
            }
        }

    }
}
