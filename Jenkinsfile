pipeline {
    agent { label 'linux docker' }
    environment {
        REPO_NAME = 'devsecops-pipeline-app'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_URI = "730335183376.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
        SONAR_PROJECT_KEY = "my-devsecops-app"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github-pat', url: "https://github.com/heyamay/Jenkins-DevSecOps-CICD-pipeline.git"
            }
        }
        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source=. --report-path=gitleaks-report.json || true'
                archiveArtifacts 'gitleaks-report.json'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.sources=.'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REPO_NAME}:latest ."
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format json -o trivy-report.json ${REPO_NAME}:latest || true"
                archiveArtifacts 'trivy-report.json'
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URI}"
                    sh "docker tag ${REPO_NAME}:latest ${ECR_REPO_URI}:latest"
                    sh "docker push ${ECR_REPO_URI}:latest"
                }
            }
        }
        stage('Archive to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh "aws s3 cp trivy-report.json s3://amay-devsecops-artifact-bucket/builds/${BUILD_NUMBER}/ --region ${AWS_REGION}"
                }
            }
        }
    }
}
