pipeline {
    agent any

    environment {
        PROJECT_ID = 'devops-demo-260815'
        CLUSTER_NAME = 'kubernetes-instance'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'Kubernetes-Demo'
    }

    stages {
    
    stage('Build Application') {
            steps {
                sh 'gradle clean build'
            }
            post {
                success {
                    echo "Now Archiving the Artifacts...."
                    archiveArtifacts artifacts: 'build/libs/*.war'
                }
            }
        }
        
         stage('Create Tomcat Docker Image'){
            steps {
                script {
                    webdemo = docker.build("srinivasan84/tomcatsampleweb:${env.BUILD_ID}")
                }
            }
        }

        stage("Image Push to Docker") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'Docker-Hub') {
                            webdemo.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps{
			    echo "Kubernetes Deployment Start"
				sh 'ls -ltr'
				sh 'pwd'
                sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Kubernetes Deployment Completed"
            }
        }        

    }
}