pipeline {
    agent any
    environment {
        //be sure to replace "amarjot16" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "amarjot16/train-schedule"
    }
    stages {
        stage('clone repo') {
	         steps {
                // step1
                echo 'clone......'
                    git url: 'https://github.com/amarjot16/cicd-pipeline-train-schedule-autodeploy', branch: 'master'
                    sh script: 'cd  $WORKSPACE'
           }		
        }
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
          //  when {
             //   branch 'master'
         //   }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
           // when {
           //     branch 'master'
         //   }
            steps {
                script {
                   withDockerRegistry(credentialsId: 'DOCKER_HUB_LOGIN', url: 'https://index.docker.io/v1/')  {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
         //   when {
          //      branch 'master'
       //     }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
           // when {
           //     branch 'master'
          //  }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
