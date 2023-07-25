pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Clonar el repositorio de Git en el directorio de trabajo del agente
                checkout scm
            }
        }

        stage('Build and Deploy') {
            environment {
                // Define las variables de entorno para el despliegue
                // Asegúrate de reemplazar los valores con los de tu clúster OpenShift
                OPENSHIFT_API_URL = 'https://api.sandbox-m3.1530.p1.openshiftapps.com:6443'
                OPENSHIFT_TOKEN = 'sha256~1qcyg2e-TekWQxE67c6CWNZ3WEogZItPuKpuM3Mn6QM'
                OPENSHIFT_NAMESPACE = 'granderamirez-6-dev'
                APPLICATION_NAME = 'core-in-open-shift-app'
                EXISTING_IMAGE_NAME = "default-route-openshift-image-registry.apps.sandbox-m3.1530.p1.openshiftapps.com/granderamirez-6-dev/core-in-open-shift:latest"
            }
            steps {
                script {
                    // Iniciar sesión en el clúster OpenShift
                    sh "oc login ${OPENSHIFT_API_URL} --token=${OPENSHIFT_TOKEN}"

                    // Check if the build configuration already exists
                    def buildConfigExists = sh(script: "oc get bc/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found=true -o name", returnStatus: true)
                    if (buildConfigExists == 0) {
                        // If the build configuration exists, start a new build
                        sh "oc start-build ${APPLICATION_NAME} --from-dir=. -n ${OPENSHIFT_NAMESPACE}"
                    } else {
                        // If the build configuration does not exist, create a new one
                        sh "oc new-build --name=${APPLICATION_NAME} --strategy=source --code=. --image-stream=dotnet:7.0-ubi8 -n ${OPENSHIFT_NAMESPACE}"
                    }

                    // Deploy the application
                    sh "oc new-app ${EXISTING_IMAGE_NAME} --name=${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    sh "oc expose service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                }
            }
        }
    }
}
