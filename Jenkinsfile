pipeline {
    agent any
    environment {
        SERVICE = 'product'
        JOB_NAME = "store-${SERVICE}"
        IMAGE_NAME = "iancdesponds/${SERVICE}"
    }
    stages {
        stage('Dependencies') {
            steps {
                build job: 'store-auth', wait: true
                build job: 'store-account', wait: true
                build job: 'store-product', wait: true
            }
        }

        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }      

        stage('Build & Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credential', usernameVariable: 'USERNAME', passwordVariable: 'TOKEN')]) {
                    sh "echo $TOKEN | docker login -u $USERNAME --password-stdin"
                    sh "docker buildx create --use --platform=linux/arm64,linux/amd64 --node multi-platform-builder-${SERVICE} --name multi-platform-builder-${SERVICE}"
                    sh "docker buildx build --platform=linux/arm64,linux/amd64 --push --tag ${IMAGE_NAME}:latest --tag ${IMAGE_NAME}:${BUILD_ID} -f Dockerfile ."
                    sh "docker buildx rm --force multi-platform-builder-${SERVICE}"
                }
            }
        }
    }
}
