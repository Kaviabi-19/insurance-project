pipeline {
    agent { label 'slave1'} 

    environment {
        MAVEN_HOME = tool name: 'Maven 3.8.7', type: 'maven'
        DOCKER_HOME = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        TAG_NAME = "3.0"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from GitHub...'
                git 'https://github.com/Kaviabi-19/insurance-project.git'
            }
        }

        stage('Build Application') {
            steps {
                echo "Building with Maven..."
                sh "${MAVEN_HOME}/bin/mvn clean package"
            }
        }

        stage('Publish Test Reports') {
            steps {
                echo "Publishing test reports..."
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'index.html',
                    reportName: 'HTML Report'
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Creating Docker image..."
                sh "${DOCKER_HOME}/docker build -t kavipriyabai/insure-me:${TAG_NAME} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Pushing Docker image to DockerHub..."
                withCredentials([usernamePassword(credentialsId: 'dockerloginPAT', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    sh "docker push kavipriyabai/insure-me:${TAG_NAME}"
                    
                }
            }
        }
        
        stage('Deploy to Test Server') {
            agent { label 'slave1' } 
            steps {
                echo "Deploying to test server using Ansible..."
                ansiblePlaybook(
                    playbook: 'ansible-playbook.yml',
                    inventory: 'ansible/hosts',
                    credentialsId: 'ansible-key',
                    become: true,
                    disableHostKeyChecking: true,
                    installation: 'ansible'   
                )
            }
        }
    }

    post {
        failure {
            emailext(
                to: 'kavipriyabai388@gmail.com',
                subject: "Job '${JOB_NAME}' #${BUILD_NUMBER} Failed",
                body: """
                Dear All,<br><br>
                The Jenkins job <b>${JOB_NAME}</b> has failed.<br>
                <a href="${BUILD_URL}">Click here to view the job</a><br><br>
                Regards,<br>Jenkins
                """,
                mimeType: 'text/html'
            )
        }
    }
}
