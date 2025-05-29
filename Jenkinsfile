pipeline {
    agent any

    environment {
        MAVEN_HOME = tool name: 'maven'
        DOCKER_HOME = tool name: 'docker'
        TAG = "3.0"
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    env.MAVEN_CMD = "${MAVEN_HOME}/bin/mvn"
                    env.DOCKER_CMD = "${DOCKER_HOME}/bin/docker"
                }
                echo "Environment prepared"
            }
        }

        stage('Git Code Checkout') {
            steps {
                script {
                    try {
                        echo 'Cloning code from GitHub'
                        git 'https://github.com/rehan282002/insureme.git'
                    } catch (Exception e) {
                        echo 'Error during Git checkout'
                        currentBuild.result = 'FAILURE'
                        emailext(
                            subject: "Job ${env.JOB_NAME} - Build #${env.BUILD_NUMBER} Failed",
                            body: """Dear All,<br>
                                The Jenkins job ${env.JOB_NAME} has failed. Please check: 
                                <a href="${env.BUILD_URL}">${env.BUILD_URL}</a>""",
                            to: 'shubham@gmail.com'
                        )
                        error("Stopping pipeline due to checkout failure")
                    }
                }
            }
        }

        stage('Build the Application') {
            steps {
                echo "Running Maven build"
                sh "${MAVEN_CMD} clean package"
            }
        }

        stage('Publish Test Reports') {
            steps {
                publishHTML([
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'index.html',
                    reportName: 'HTML Report'
                ])
            }
        }

        stage('Containerize the Application') {
            steps {
                echo "Building Docker Image"
                sh "${DOCKER_CMD} build -t rehan282002/insureme:${TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Pushing image to DockerHub"
                withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
                    sh "${DOCKER_CMD} login -u rehan282002 -p ${dockerHubPassword}"
                    sh "${DOCKER_CMD} push rehan282002/insureme:${TAG}"
                }
            }
        }

        stage('Deploy to Server using Ansible') {
            steps {
                echo "Running Ansible Playbook"
                ansiblePlaybook(
                    become: true,
                    credentialsId: 'ansible-key',
                    disableHostKeyChecking: true,
                    installation: 'ansible',
                    inventory: '/etc/ansible/hosts',
                    playbook: 'ansible-playbook.yml'
                )
            }
        }
    }
}
