pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Clonar el repositorio de Git en el directorio de trabajo del agente
                checkout scm
            }
        }

        stage('Deploy') {
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

                    // Delete the existing service if it already exists
                    sh "oc delete service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found"

                    // Update the existing deployment configuration with the latest image
                    sh "oc set image deploymentconfig/${APPLICATION_NAME} ${APPLICATION_NAME}=${EXISTING_IMAGE_NAME} -n ${OPENSHIFT_NAMESPACE}"

                    // Expose the service
                    sh "oc expose dc ${APPLICATION_NAME} --port=80 -n ${OPENSHIFT_NAMESPACE}"

                    // Patch the existing route with the latest changes
                    sh "oc patch route ${APPLICATION_NAME} -p '{\"spec\":{\"to\":{\"name\":\"${APPLICATION_NAME}\"}}}' -n ${OPENSHIFT_NAMESPACE}"
                }
            }
        }
    }
}
