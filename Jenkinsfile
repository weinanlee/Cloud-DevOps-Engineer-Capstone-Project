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

        stage('Create cluster configuration') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        aws eks --region us-west-2 update-kubeconfig --name capstone-cloud-devops-cluster
                    '''
                }
            }
        }

        stage('Set kubectl context') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        kubectl config use-context arn:aws:eks:us-west-2:918317714877:cluster/capstone-cloud-devops-cluster
                    '''
                }
            }
        }


        stage('Deploy blue container') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        kubectl apply -f ./blue-controller.yml
                    '''
                }
            }
        }

        stage('Deploy green container') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        kubectl apply -f ./green-controller.yml
                    '''
                }
            }
        }


        stage('Create service in cluster, redirect to green') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        kubectl apply -f green-service.yml
                    '''
                }
            }
        }

        stage('Wait user approve') {
            steps {
                input "The service will redict to blue..."
            }
        }

        stage('Create service in cluster, redirect to blue') {
            steps {
                withAWS(region:'us-west-2', credentials:'aws-devops') {
                    sh '''
                        kubectl apply -f blue-service.yml
                    '''
                }
            }
        } 


    }
}
