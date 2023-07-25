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
                    // Check if the DeploymentConfig exists
                    def dcExists = sh(returnStatus: true, script: "oc get dc ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --no-headers")

                    if (dcExists == 0) {
                        echo "DeploymentConfig ${APPLICATION_NAME} already exists. Reusing the existing DeploymentConfig."
                    } else {
                        // Create the DeploymentConfig if it does not exist
                        sh "oc new-app ${EXISTING_IMAGE_NAME} --name=${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    }

                    // Check if the service exists
                    def serviceExists = sh(returnStatus: true, script: "oc get service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --no-headers")

                    if (serviceExists == 0) {
                        echo "Service ${APPLICATION_NAME} already exists. Reusing the existing service."
                    } else {
                        // Create the service if it does not exist
                        sh "oc expose dc ${APPLICATION_NAME} --port=80 -n ${OPENSHIFT_NAMESPACE}"
                    }

                    // Check if the route exists
                    def routeExists = sh(returnStatus: true, script: "oc get route ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --no-headers")

                    if (routeExists == 0) {
                        echo "Route ${APPLICATION_NAME} already exists. Reusing the existing route."
                    } else {
                        // Patch the route if it does not exist
                        sh "oc patch route ${APPLICATION_NAME} -p '{\"spec\":{\"to\":{\"name\":\"${APPLICATION_NAME}\"}}}' -n ${OPENSHIFT_NAMESPACE}"
                    }
                }
            }
        }
    }
}
