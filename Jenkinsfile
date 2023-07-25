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
                IMAGE_STREAM_TAG = 'latest' // Use a specific image stream tag here
                IMAGE_NAME = "default-route-openshift-image-registry.apps.sandbox-m3.1530.p1.openshiftapps.com/granderamirez-6-dev/core-in-open-shift-app:${IMAGE_STREAM_TAG}"
            }
            steps {
                script {
                    // Iniciar sesión en el clúster OpenShift
                    sh "oc login ${OPENSHIFT_API_URL} --token=${OPENSHIFT_TOKEN}"

                    // Create a new build and deploy the application if the image stream tag is not available
                    def imageStreamTagExists = sh(script: "oc get istag/${IMAGE_STREAM_TAG} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found=true -o name", returnStatus: true)
                    if (imageStreamTagExists != 0) {
                        sh "oc new-build --name=${APPLICATION_NAME} --binary --strategy=source --code=." // Build the source code into a new image
                        sh "oc start-build ${APPLICATION_NAME} --from-dir=. --follow -n ${OPENSHIFT_NAMESPACE}"
                        sh "oc tag ${APPLICATION_NAME}:latest ${IMAGE_NAME}"
                        sh "oc new-app ${IMAGE_NAME} --name=${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    }

                    // Expose the service if it doesn't exist
                    def serviceStatus = sh(returnStatus: true, script: "oc get service/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --no-headers")
                    if (serviceStatus != 0) {
                        sh "oc expose service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    }
                }
            }
        }
    }
}
