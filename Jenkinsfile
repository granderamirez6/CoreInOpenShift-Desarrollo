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
                OPENSHIFT_TOKEN = 'sha256~drvh2PNAdGNV3ezjjwpQtY3iVg5MetQhoeEGmcrHoG0'
                OPENSHIFT_NAMESPACE = 'granderamirez-6-dev'
                APPLICATION_NAME = 'core-in-open-shift-app'
                EXISTING_IMAGE_NAME = "${OPENSHIFT_API_URL}/granderamirez-6-dev/${APPLICATION_NAME}"
            }
            steps {
                script {
                    // Iniciar sesión en el clúster OpenShift
                    sh "oc login ${OPENSHIFT_API_URL} --token=${OPENSHIFT_TOKEN}"

                    // Desplegar la aplicación en el clúster
                    sh "oc new-app ${EXISTING_IMAGE_NAME} --name=${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    sh "oc expose service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                }
            }
        }
    }
}
