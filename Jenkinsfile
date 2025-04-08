pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        S3_BUCKET = 'autoprovision'
        ZIP_NAME = 'system-logger.zip'
        PY_SCRIPT_PATH = 'roles/python_script/files/log_stats.py'
    }

    options {
        timestamps()
        skipStagesAfterUnstable()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bodukal/Auto-provision.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    sudo apt update
                    sudo apt install -y python3-pip zip curl unzip python3-venv python3-full

                    python3 -m venv venv
                    . venv/bin/activate

                    pip install boto3 ansible awscli
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'SSH_Credentials', keyFileVariable: 'PRIVATE_KEY')]) {
                    sh '''
                        export ANSIBLE_HOST_KEY_CHECKING=False
                        . venv/bin/activate

                        ansible-playbook -i hosts.ini deploy_app.yml \
                          --extra-vars "artifact_url=https://s3.amazonaws.com/autoprovision/system-logger.zip" \
                          --private-key $PRIVATE_KEY
                    '''
                }
            }
        }

        stage('Archive Python Script') {
            steps {
                sh '''
                    mkdir -p artifacts
                    zip -j artifacts/${ZIP_NAME} ${PY_SCRIPT_PATH}
                '''
                archiveArtifacts artifacts: "artifacts/${ZIP_NAME}", fingerprint: true
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '63441d18-32d3-4987-907e-489b2ef3b56c']]) {
                    sh '''
                        source venv/bin/activate
                        aws s3 cp artifacts/${ZIP_NAME} s3://${S3_BUCKET}/${ZIP_NAME} --region ${AWS_REGION}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed and artifact uploaded to S3!"
        }
        failure {
            echo "❌ Pipeline failed. Please check logs."
        }
    }
}
