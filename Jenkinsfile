
pipeline {
    agent any

    environment {
        IMAGE_NAME = '441870321480.dkr.ecr.us-east-1.amazonaws.com/test-image-vuln-scan-base'
        IMAGE_TAG = 'latest'
	GIT_COMMIT_HASH = sh (script: "git rev-parse --short `git log -n 1 --pretty=format:'%H'`", returnStdout: true)
    }

    stages {
        stage('Prepare') {
            steps {
                echo 'Initializing submodules..'
                sh 'ln -s local.env .env'
                sh 'git submodule update --init --recursive'
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'docker build . -t ${IMAGE_NAME}:ci'
		sh 'docker push ${IMAGE_NAME}:ci'
            }
        }
        stage('Scan') {
            steps {
		echo 'Scanning..'
//                sh 'curl -s https://ci-tools.anchore.io/inline_scan-v0.3.3 | bash -s -- ${IMAGE_NAME}:ci'
		writeFile file: "anchore_images", text: "${IMAGE_NAME}:ci"
	        anchore name: "anchore_images", engineurl: <anchore-engine-url>, engineCredentialsId: <jenkins-credentials-id>
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
//                junit '**/target/*.xml'
            }
        }
        stage('QA') {
            steps {
                echo 'Checking code..'
            }
        }

        stage('Push Image') {
            steps {
            	sh '$(aws ecr get-login --no-include-email)'
		sh 'docker tag ${IMAGE_NAME}:ci ${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
		sh 'docker tag ${IMAGE_NAME}:ci ${IMAGE_NAME}:${GIT_COMMIT_HASH}'
                sh 'docker push ${IMAGE_NAME}:${GIT_COMMIT_HASH}'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}

node {
  def imageLine = '441870321480.dkr.ecr.us-east-1.amazonaws.com/test-image-vuln-scan-base:ci'
  writeFile file: 'anchore_images', text: imageLine
  anchore name: 'anchore_images'
}
