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
                // Define las variables de entorno para la construcción y el despliegue
                // Asegúrate de reemplazar los valores con los de tu clúster OpenShift
                OPENSHIFT_API_URL = 'https://api.sandbox-m3.1530.p1.openshiftapps.com:6443'
                OPENSHIFT_TOKEN = 'sha256~1qcyg2e-TekWQxE67c6CWNZ3WEogZItPuKpuM3Mn6QM'
                OPENSHIFT_NAMESPACE = 'granderamirez-6-dev'
                APPLICATION_NAME = 'granderamirez-6-dev'
                SOURCE_REPOSITORY = 'https://github.com/granderamirez6/CoreInOpenShift'
            }
            steps {
                script {
                    // Construir la imagen del contenedor utilizando S2I y .NET Core
                    def customImageName = "${OPENSHIFT_API_URL}/your-openshift-project/${APPLICATION_NAME}"
                    sh "oc login ${OPENSHIFT_API_URL} --token=${OPENSHIFT_TOKEN}"
                    sh "oc new-build --name=${APPLICATION_NAME} --binary --strategy=source ${SOURCE_REPOSITORY} -n ${OPENSHIFT_NAMESPACE}"
                    sh "oc start-build ${APPLICATION_NAME} --from-dir=. --follow -n ${OPENSHIFT_NAMESPACE}"
                    sh "oc tag ${APPLICATION_NAME}:latest ${customImageName}:latest"

                    // Desplegar la aplicación en el clúster
                    sh "oc new-app ${customImageName}:latest --name=${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    sh "oc expose service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                }
            }
        }
    }
}
