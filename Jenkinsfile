pipeline {
    agent any

    environment {
        IMAGE = "ghcr.io/rkstechforge/stackpulse"
    }

    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Build Java") {
            steps {
                sh 'mvn -B -f deployment/StackPulseAwsDeployment/pom.xml test package'
            }
        }

        stage("Docker Build") {
            steps {
                sh '''
                    docker build                       -t "$IMAGE:$BUILD_NUMBER"                       -t "$IMAGE:latest"                       -f deployment/StackPulseAwsDeployment/applications/stackpulse/Dockerfile                       deployment/StackPulseAwsDeployment/applications/stackpulse
                '''
            }
        }

        stage("Push") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "GHCR_CREDENTIALS",
                    usernameVariable: "CR_USER",
                    passwordVariable: "CR_TOKEN"
                )]) {
                    sh '''
                        echo "$CR_TOKEN" | docker login ghcr.io -u "$CR_USER" --password-stdin
                        docker push "$IMAGE:$BUILD_NUMBER"
                        docker push "$IMAGE:latest"
                    '''
                }
            }
        }

        stage("Deploy") {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: "DEPLOY_SSH",
                    keyFileVariable: "KEY",
                    usernameVariable: "USER"
                )]) {
                    sh '''
                        ssh -i "$KEY" "$USER@$DEPLOY_HOST"                           "docker pull $IMAGE:latest &&                            docker rm -f stackpulse 2>/dev/null || true &&                            docker run -d --name stackpulse --restart unless-stopped -p 8080:8080 $IMAGE:latest"
                    '''
                }
            }
        }
    }
}
