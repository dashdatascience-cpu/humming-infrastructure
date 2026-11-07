pipeline {
    agent {
        label 'humming-node24'
    }

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare GKE Tools') {
            steps {
                container('gcloud') {
                    sh '''
                        set -eu

                        export DEBIAN_FRONTEND=noninteractive

                        echo "Installing Kubernetes tools..."
                        apt-get update
                        apt-get install -y \
                          kubectl \
                          google-cloud-cli-gke-gcloud-auth-plugin

                        echo "Connecting to Humming GKE cluster..."
                        gcloud container clusters get-credentials \
                          humming-cluster \
                          --region=asia-south1 \
                          --project=project-3ae33600-7198-48d4-a61

                        echo "Verifying Kubernetes connection..."
                        kubectl config current-context
                    '''
                }
            }
        }

        stage('Validate Configuration') {
            steps {
                container('gcloud') {
                    sh '''
                        echo "Checking JSON validators..."
                        command -v python3 || true
                        command -v jq || true
                        command -v node || true
                    '''
                    sh '''
                        set -eu

                        echo "Validating Traefik JSON manifests..."

                        for file in \
                          kubernetes/traefik/*-route.json \
                          kubernetes/traefik/*-routes.json \
                          kubernetes/traefik/*-middleware.json; do

                            [ -f "$file" ] || continue

                            echo "Checking $file"
                            jq empty "$file"
                        done

                        echo "Validation completed."
                    '''
                }
            }
        }

        stage('Approve Deployment') {
            steps {
                input message: 'Deploy Humming Traefik routes to GKE?',
                      ok: 'Deploy'
            }
        }

        stage('Deploy Traefik Routes') {
            steps {
                container('gcloud') {
                    sh '''
                        set -eu

                        for file in \
                          kubernetes/traefik/*-middleware.json \
                          kubernetes/traefik/*-route.json \
                          kubernetes/traefik/*-routes.json; do

                            [ -f "$file" ] || continue

                            echo "Applying $file"
                            kubectl apply -f "$file"
                        done
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                container('gcloud') {
                    sh '''
                        kubectl get ingressroutes,middlewares \
                          -n jenkins
                    '''
                }
            }
        }
    }
}
