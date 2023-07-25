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
                SOURCE_REPOSITORY = 'https://github.com/granderamirez6/CoreInOpenShift'
            }
            steps {
                script {
                    // Iniciar sesión en el clúster OpenShift
                    sh "oc login ${OPENSHIFT_API_URL} --token=${OPENSHIFT_TOKEN}"

                    // Check if the BuildConfig exists
                    def buildConfigExists = sh(script: "oc get bc/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found=true -o name", returnStatus: true)

                    if (buildConfigExists != 0) {
                        // Create a new BuildConfig for the application
                        sh "oc new-build --name=${APPLICATION_NAME} --binary --strategy=source --code=. --image-stream=dotnet:5.0 -n ${OPENSHIFT_NAMESPACE}"
                    }

                    // Start the build only if the BuildConfig exists
                    def buildStatus = sh(returnStatus: true, script: "oc get build/${APPLICATION_NAME}-1 -n ${OPENSHIFT_NAMESPACE} --no-headers")
                    if (buildStatus != 0) {
                        sh "oc start-build ${APPLICATION_NAME} --from-dir=. --follow -n ${OPENSHIFT_NAMESPACE}"
                    } else {
                        echo "Build for ${APPLICATION_NAME} is already in progress or completed."
                    }

                    // Tag the new image with the specified image stream tag
                    sh "oc tag ${APPLICATION_NAME}:${IMAGE_STREAM_TAG} ${APPLICATION_NAME}:${IMAGE_STREAM_TAG} -n ${OPENSHIFT_NAMESPACE}"

                    // Create a new application or update the existing one
                    def existingDeployment = sh(script: "oc get dc/${APPLICATION_NAME} -n ${OPENSHIFT_NAMESPACE} --ignore-not-found=true -o name", returnStdout: true).trim()
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
