pipeline{
    agent any 
    tools {
        jdk 'Java21'
        maven 'Maven'
    }
    environment {
        APP_NAME = "complete-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "yasindevops06"
        // DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', url: 'https://github.com/yasindevops/error-cont-jenkins.git'
            }

        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }
        
        // stage("Sonarqube Analysis") {
        //     steps {
        //         script {
        //             withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
        //                 sh "mvn sonar:sonar"
        //             }
        //         }
        //     }

        // }

        // stage("Quality Gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
        //         }
        //     }

        // }

        stage("Code Coverage with JaCoCo") {
            steps {
                sh "mvn jacoco:prepare-agent test jacoco:report"
                archiveArtifacts 'target/site/jacoco/index.html'
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Retrieve DockerHub token from Jenkins credentials
                    withCredentials([string(credentialsId: 'my-dockerhub-token', variable: 'DOCKERHUB_TOKEN')]) {
                        
                        def dockerRegistryUrl = 'https://index.docker.io/v1/'

                        // Build the Docker image
                        docker.withRegistry(dockerRegistryUrl, DOCKERHUB_TOKEN) {
                            docker_image = docker.build "${IMAGE_NAME}"
                        }

                        // Push the Docker image to DockerHub
                        docker.withRegistry(dockerRegistryUrl, DOCKERHUB_TOKEN) {
                            docker_image.push("${IMAGE_TAG}")
                            docker_image.push('latest')
                        }
                    }
                }
            }
        }



        stage("Trivy Scan") {
            steps {
                script {
		   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG} --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }

        }

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }


        // stage("Trigger CD Pipeline") {
        //     steps {
        //         script {
        //             sh "curl -v -k --user yasin_devops:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.dev.dman.cloud/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
        //         }
        //     }

        // }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                def localJenkinsUrl = 'http://localhost:8080'
                def localJenkinsJobPath = 'job/gitops-complete-pipeline'
                def localJenkinsToken = 'gitops-token'
            
            sh "curl -v -k --user yasin_devops:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '${localJenkinsUrl}/${localJenkinsJobPath}/buildWithParameters?token=${localJenkinsToken}'"
        }
    }
}


    }

    post {
        failure {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                    mimeType: 'text/html',to: "yasinhasturk@hotmail.com"
            }
         success {
               emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                    mimeType: 'text/html',to: "yasinhasturk@hotmail.com"
          }      
    }
}
