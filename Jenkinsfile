// Jenkins pipeline to deploy simple-web
// Prerequisites:
// - Kubernetes CLI Plugin installed
// - "kubeconfig" credentialId with proper config configured

pipeline {
    agent any //@FIXME use worker node labels to pick workers with helm installed and k8s access configured.
    
    parameters {
        choice(choices: ['install/upgrade', 'delete'], name: 'action')
    }
    
    stages {
        stage('Deploy or delete from cluster') {
            steps {
                //concurrency control
                lock('kubernetes-cluster') {
                    withKubeConfig([credentialsId: 'kubeconfig',
                                    //serverUrl: 'https://kubernetes.cluster.local', //@FIXME serverUrl is optional
                                    namespace: 'tomasz']) {
                        if (params.action == 'install') {
                            //add a repository or silentyluy fail if already added
                            sh 'helm repo add simple-web https://chieftainy2k.github.io/helm-charts-simple-web || true'
                            //update repository
                            sh 'helm repo update'
                            //upgrade or install the app
                            sh 'helm upgrade --install --wait --timeout simple-web simple-web/simple-web'
                            //test the deployment
                            sh 'helm test simple-web'
                        } else if (params.action == 'delete') {
                            //delete the app
                            sh 'helm uninstall --wait simple-web'
                        }
                    }
                }
            }
        }
    }
}