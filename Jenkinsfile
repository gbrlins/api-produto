pipeline {
    agent any

    stages {
        stage ('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("gabriellins/api-produto:${env.BUILD_ID}", '-f ./src/Dockerfile ./src') 
                }                
            }
        }

        stage ('Verificar imagem na registry temporaria com Neuvector') {
            steps {
                script {      
                    neuvector nameOfVulnerabilityToExemptFour: '', nameOfVulnerabilityToExemptOne: '', nameOfVulnerabilityToExemptThree: '', nameOfVulnerabilityToExemptTwo: '', nameOfVulnerabilityToFailFour: '', nameOfVulnerabilityToFailOne: '', nameOfVulnerabilityToFailThree: '', nameOfVulnerabilityToFailTwo: '', numberOfHighSeverityToFail: '1', numberOfMediumSeverityToFail: '', registrySelection: 'Docker-hub', repository: 'gabriellins/api-produto', scanLayers: true, scanTimeout: 10, tag: 'latest'
                }
            }
        }
        
        stage ('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage ('Deploy Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'
                }
            }
        }
    }
}
