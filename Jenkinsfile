pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        stage('compile') {
            agent { label 'knode1'}
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            agent { label 'knode1'}
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'mvn package'
            }
        }
        stage('copy') {
            steps {
                ansiblePlaybook credentialsId: 'knode1', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: '/var/lib/jenkins/workspace/pipeline/copy.yaml', vaultTmpPath: ''
            }
        }
        stage('ssh-connect') {
            steps {
               script {
                   sshagent(credentials: ['knode1'], ignoreMissing: true) {
                sh ''' 
                ssh -n -o StrictHostKeyChecking=no edureka@172.31.19.27 '
                cd /home/edureka && \
                docker build -t jacekcracow/abctechnologies:v1 . && \
                docker push jacekcracow/abctechnologies:v1
               '
              '''
              }
            }

            }
        }
        stage('Deploy to Kubernetes') {
    steps {
        script {
            kubeconfig(credentialsId: 'kubernetes', serverUrl: 'https://172.31.19.27:6443/') {
                sh '''
                kubectl apply -f project_required_file_v2/deployment.yml --namespace=jenkins --validate=false
                '''
            }
        }
    }
}
}
}
