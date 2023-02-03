// Jenkins pipeline to deploy simple-web
// Prerequisites:
// - Kubernetes CLI Plugin installed
// - "kubeconfig" credentialId with proper config configured

pipeline {
    agent any //@FIXME use worker node labels to pick workers with helm installed and k8s access configured.

    parameters {
        choice(choices: ['install', 'delete'], name: 'action')
    }

    options {
        //prevent from concurrent runs (alternatively use lock(resource: 'simple-web-app-deployment') for more control)
        disableConcurrentBuilds()
    }

    stages {
        stage('Add Helm repository') {
            when {
                expression { params.action == 'install' }
            }
            steps {
                //add a repository , ignore errors at this point (such as repo already added)
                sh 'helm repo add simple-web https://chieftainy2k.github.io/helm-charts-simple-web || true'
            }
        }
    }

    stages {
        stage('Deploy from cluster') {
            when {
                expression { params.action == 'install' }
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig', namespace: 'tomasz']) {
                    sh 'helm upgrade --install simple-web simple-web/simple-web --wait'
                }
            }
        }

        stage('Test the app') {
            when {
                expression { params.action == 'install' }
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig', namespace: 'tomasz']) {
                    sh 'helm test simple-web --wait'
                }
            }
        }

        stage('Delete from cluster') {
            when {
                expression { params.action == 'delete' }
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig', namespace: 'tomasz']) {
                    sh 'helm delete simple-web --wait'
                }
            }
        }
    }
}