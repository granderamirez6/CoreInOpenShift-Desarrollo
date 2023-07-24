pipeline {
    agent any
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
                sh 'dotnet build -c Release'
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
