@Library('smartPipeline@main') _

pipeline {
    agent any
    stages {
        stage('Build & Test') {
            steps {
                smartBuild(type: 'maven', pom: 'deployment/StackPulseAwsDeployment/pom.xml', goals: 'test package')
            }
        }
        stage('Docker Build') {
            steps {
                smartContainer(
                    image: "ghcr.io/rkstechforge/stackpulse:${env.BUILD_NUMBER}",
                    dockerfile: 'deployment/StackPulseAwsDeployment/applications/stackpulse/Dockerfile',
                    context: 'deployment/StackPulseAwsDeployment/applications/stackpulse',
                    snykScan: false
                )
                sh 'docker tag ghcr.io/rkstechforge/stackpulse:$BUILD_NUMBER ghcr.io/rkstechforge/stackpulse:latest'
            }
        }
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'GHCR_CREDENTIALS', usernameVariable: 'CR_USER', passwordVariable: 'CR_TOKEN')]) {
                    sh '''
                        echo "$CR_TOKEN" | docker login ghcr.io -u "$CR_USER" --password-stdin
                        docker push "ghcr.io/rkstechforge/stackpulse:$BUILD_NUMBER"
                        docker push "ghcr.io/rkstechforge/stackpulse:latest"
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'DEPLOY_SSH', keyFileVariable: 'KEY', usernameVariable: 'USER')]) {
                    sh '''
                        ssh -i "$KEY" "$USER@$DEPLOY_HOST"                           "docker pull ghcr.io/rkstechforge/stackpulse:latest &&
                           docker rm -f stackpulse 2>/dev/null || true &&
                           docker run -d --name stackpulse --restart unless-stopped -p 8080:8080 ghcr.io/rkstechforge/stackpulse:latest"
                    '''
                }
            }
        }
    }
}
