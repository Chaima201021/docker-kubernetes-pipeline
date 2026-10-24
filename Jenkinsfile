pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'chaima14/fleetman-webapp:latest'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Checkout') {
            steps {
                retry(3) {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/master']],
                        extensions: [[$class: 'CloneOption', depth: 1, shallow: true, noTags: true]],
                        userRemoteConfigs: [[url: 'https://github.com/Chaima201021/docker-kubernetes-pipeline.git']]
                    ])
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=fleetman-webapp \
                              -Dsonar.projectName=Fleetman-Webapp \
                              -Dsonar.sources=dist \
                              -Dsonar.exclusions=**/*.js \
                              -Dsonar.sourceEncoding=UTF-8
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            -u "$DOCKERHUB_USER" --password-stdin
                        docker push "$DOCKER_IMAGE"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl --kubeconfig="$KUBECONFIG" \
                        set image deployment/webapp \
                        webapp="$DOCKER_IMAGE"
                    kubectl --kubeconfig="$KUBECONFIG" \
                        rollout status deployment/webapp --timeout=180s
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline terminé avec le statut : ${currentBuild.currentResult}"
        }
        failure {
            echo "Le pipeline a échoué. Consultez la Console Output pour le détail."
        }
    }
}
