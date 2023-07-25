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
                OPENSHIFT_NAMESPACE = 'granderamirez-6-dev'
                APPLICATION_NAME = 'core-in-open-shift-app'
                EXISTING_IMAGE_NAME = "image-registry.openshift-image-registry.svc:port/granderamirez-6-dev/core-in-open-shift-app:latest"
            }
            steps {
                script {
                    // Delete the existing service if it already exists
                    sh "oc delete service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found"

                    // Get the container name from the deployment configuration
                    def containerName = sh(script: "oc get dc/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} -o jsonpath='{.spec.template.spec.containers[0].name}'", returnStdout: true).trim()

                    // Update the existing deployment configuration with the latest image
                    sh "oc set image dc/${APPLICATION_NAME} ${containerName}=${EXISTING_IMAGE_NAME} -n ${OPENSHIFT_NAMESPACE}"

                    // Expose the service
                    sh "oc expose dc ${APPLICATION_NAME} --port=80 -n ${OPENSHIFT_NAMESPACE}"

                    // Patch the existing route with the latest changes
                    sh "oc patch route ${APPLICATION_NAME} -p '{\"spec\":{\"to\":{\"name\":\"${APPLICATION_NAME}\"}}}' -n ${OPENSHIFT_NAMESPACE}"
                }
            }
        }
    }
}
