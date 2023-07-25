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
                IMAGE_STREAM_TAG = 'latest' // Utiliza la etiqueta de la imagen deseada
                SOURCE_REPOSITORY = 'https://github.com/granderamirez6/CoreInOpenShift'
            }
            steps {
                script {
                    // Iniciar sesión en el clúster OpenShift
                    sh "oc login ${OPENSHIFT_API_URL} --token=${OPENSHIFT_TOKEN}"

                    // Verificar si la imagen existe en el namespace y obtener el ID de la imagen
                    def existingImageId = sh(script: "oc get is/${APPLICATION_NAME}:${IMAGE_STREAM_TAG} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found=true -o jsonpath='{.status.dockerImageRepository}'", returnStdout: true).trim()

                    // Si la imagen no existe, crear una nueva BuildConfig y realizar una nueva construcción
                    if (!existingImageId) {
                        sh "oc new-build --name=${APPLICATION_NAME} --binary --strategy=source --code=. --image-stream=dotnet:5.0 -n ${OPENSHIFT_NAMESPACE}"
                        sh "oc start-build ${APPLICATION_NAME} --from-dir=. --follow -n ${OPENSHIFT_NAMESPACE}"
                        sh "oc tag ${APPLICATION_NAME}:latest ${APPLICATION_NAME}:${IMAGE_STREAM_TAG} -n ${OPENSHIFT_NAMESPACE}"
                    } else {
                        echo "La imagen ya existe, se utilizará la imagen existente."
                    }

                    // Desplegar la aplicación utilizando la imagen existente
                    def existingDeployment = sh(script: "oc get dc/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found=true -o jsonpath='{.metadata.name}'", returnStdout: true).trim()
                    if (!existingDeployment) {
                        sh "oc new-app ${APPLICATION_NAME}:${IMAGE_STREAM_TAG} --name=${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                        sh "oc expose service ${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    } else {
                        sh "oc rollout latest dc/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE}"
                    }
                }
            }
        }
    }
}
