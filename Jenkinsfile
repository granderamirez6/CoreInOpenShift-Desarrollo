pipeline {
    agent {
        docker {
            image 'your-custom-jenkins-agent:latest' // Use the custom Jenkins agent Docker image with .NET SDK
            args '-u root' // Run the container with root privileges (if required)
        }
    }
    stages {
        stage('Clonar Repositorio') {
            steps {
                checkout scm
            }
        }
        stage('Restaurar Dependencias') {
            steps {
                sh 'dotnet restore'
            }
        }
        stage('Construir') {
            steps {
                script {
                    def customImageName = "image-registry.openshift-image-registry.svc:5000/granderamirez-6-dev/core-in-open-shift"
                    // Build the Docker image
                    sh "docker build -t $customImageName ."

                    // Push the Docker image to the container registry
                    sh "docker push $customImageName"

                    // Store the image URL in an environment variable for later use
                    env.CUSTOM_IMAGE_URL = customImageName
                }
            }
        }
        stage('Desplegar en OpenShift') {
            steps {
                openshiftDeploy apiURL: 'https://core-in-open-shift-granderamirez-6-dev.apps.sandbox-m3.1530.p1.openshiftapps.com',
                    authToken: 'sha256~z3bgyhyNXaUi47gnXb7jlLdbh34NmSiPBgNPbLlc49Y',
                    depCfg: 'core-in-open-shift', 
                    namespace: 'granderamirez-6-dev',
                    verbose: true
            }
        }
    }
}
