pipeline {
    agent any
    environment {
        PROJECT_ID = 'snappy-benefit-260010'
        CLUSTER_NAME = 'cluster-kubernetes-1'
        LOCATION = 'europe-west1-b'
        CREDENTIALS_ID = 'kuberneteslogin'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
		 stage("Build") {
            steps {
               echo "cleaning and packaging"
			   sh 'mvn clean package'
            }
        }
		 stage("Test") {
            steps {
                echo "Testing"
			   sh 'mvn test'
            }
        }
        stage("Build Docker Image") {
            steps {
                script {
                    myimage = docker.build("gcr.io/snappy-benefit-260010/kubernetesrepos:${env.BUILD_ID}")
                }
            }
        }
        stage("Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('https://eu.gcr.io', 'gcr:kuberneteslogin') {
                            myimage.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
        stage('Deploy to Google Kubernetes') {
            steps{
			    echo "Deployment started"
				sh 'ls -ltr'
				sh 'pwd'
                sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Deployment Finished"
            }
        }
    }    
}
