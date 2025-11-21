pipeline {
    agent any
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build application') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Upload Artifact to S3') {
            steps {
                sh '''
                    WAR_FILE=/opt/homebrew/opt/tomcat/libexec/webapps/my-app.war
                    BUCKET=harsh-s3-maven-bucket1q2w3e4r
                    VERSION=$(date +%Y-%m-%d-%H-%M-%S)

                    
                    echo "Uploading $WAR_FILE to S3 bucket $BUCKET"
                    aws s3 cp $WAR_FILE s3://$BUCKET/NODE-artifacts/my-app.war
                '''
            }
        }
    }
}
