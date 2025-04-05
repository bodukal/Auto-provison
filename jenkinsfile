pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_INVENTORY = '/home/ubuntu/Auto-provison/hosts.ini'  // Path to your hosts.ini file
    }

    stages {
        stage('Clone Repo') {
            steps {
                // Clone the GitHub repository to get the latest project code
                git 'https://github.com/bodukal/Auto-provison.git'
            }
        }

        stage('Run SQL Setup') {
            steps {
                // Run the SQL setup playbook to install MySQL, create DB and table
                sh '''ansible-playbook -i ${ANSIBLE_INVENTORY} sql_setup.yml'''
            }
        }

        stage('Run Python Setup') {
            steps {
                // Run Python setup playbook to install Python, dependencies, and copy the script
                sh '''ansible-playbook -i ${ANSIBLE_INVENTORY} python_setup.yml'''
            }
        }

        stage('Setup Cron Job') {
            steps {
                // Run the playbook to schedule the Python script using cron or systemd
                sh '''ansible-playbook -i ${ANSIBLE_INVENTORY} cron_setup.yml'''
            }
        }

        stage('Archive and Upload Artifact') {
            steps {
                // Zip the Python script and configuration files
                sh '''zip -r script.zip /path/to/log_stats.py /path/to/configs'''
                // Upload the zipped artifact to S3 (or your preferred storage)
                sh '''aws s3 cp script.zip s3://your-bucket-name/'''
            }
        }
    }

    post {
        always {
            // Clean up or notification steps
            echo 'Pipeline execution complete.'
        }
    }
}
