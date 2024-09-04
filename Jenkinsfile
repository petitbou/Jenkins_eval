pipeline {
    agent any
    environment {
        DOCKER_USERNAME = '' 
        DOCKER_CREDENTIALS = credentials('') 
        DOCKER_IMAGE = ''
    }
    stages {
        stage('Git clone') {
            steps {
                checkout scm
            }
        }
        stage('Build and push images') { // Build image step
            when {
                anyOf {
                    branch 'dev'
                    branch 'QA'
                    branch 'staging'
                    branch 'master'
                }
            }
            steps {
                script { // Ajouter images cast et movie
                    sh "docker login -u ${env.DOCKER_USERNAME} -p ${DOCKER_CREDENTIALS}"
                    sh "docker build -t ${env.DOCKER_IMAGE}:latest -f ./Dockerfile"
                    sh "docker push ${env.DOCKER_IMAGE}:latest"
                    sh "docker logout"
                }
            }
        }        
        stage('Deploy to Environments') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'QA'
                    branch 'staging'
                }
            }
            environment {
                KUBECONFIG = credentials("config") // kubeconfig from secret from jenkins
            }
            steps {
                script {                    
                    def valuesFile = "values-${env.BRANCH_NAME}.yaml"
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config                    
                    '''
                    sh "helm upgrade --install my-release ./my-app --namespace ${env.BRANCH_NAME} --values ./my-app/${valuesFile} --set image.tag=${env.BUILD_NUMBER}"
                }
            }
        }
        stage('Deployment to Production') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config") // kubeconfig from secret from jenkins
            }
            steps {
                timeout(time: 5, unit: "MINUTES") {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    '''
                    sh "helm upgrade --install my-release ./my-app --namespace prod --values ./jenkins-charts/values-prod.yaml --set image.tag=${env.BUILD_NUMBER}"
                }
            }
        }
    }
    post {
        always {
            echo 'done.'
            cleanWs() // Clean the workspace afterward
        }
        failure {
            echo 'failure!'
        }
    }
}
