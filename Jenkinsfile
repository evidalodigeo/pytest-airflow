import java.text.SimpleDateFormat

pipeline {
    agent any
    environment {
        GOOGLE_APPLICATION_CREDENTIALS = credentials('GOOGLE_APPLICATION_CREDENTIALS_JENKINS')
        LOCATION = 'europe-west1'
    }
    parameters {
        choice(choices: ['edo-datalake-lab01', 'edo-dev-ds-datalake', 'datascience-210113'], description: 'Environment[lab|qa|prod]', name: 'GOOGLE_PROJECT_ID')
        string(name: 'RELEASE', defaultValue: '1.0.0', description: 'default version docker image. ex: 1.0.0')

    }
    stages {
        stage('Clean') {
            steps {
                echo "Cleaning ..."
                sh 'rm -rf $PROJECT_ETL_NAME'
            }
        }
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'LocalBranch', localBranch: 'master']], submoduleCfg: [], userRemoteConfigs: [[url: 'git@github.com:evidalodigeo/pytest-airflow.git']]])
            }
        }
        stage('authentication') {
            steps {
                echo 'Authentication...'
                sh 'gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}'
                sh 'gcloud config set project ${GOOGLE_PROJECT_ID}'
                sh 'gcloud info'
            }
        }
         stage('update python') {
            steps {
                sh label: '', script: '''
                    python3 --version
                    apt-get update
                    apt-get install python3-pip -y
                    python3 --version
                    pip3 install --upgrade pip setuptools wheel twine
                    '''
            }
        }
        stage('distribution package'){
            steps{
                sh "python3 setup.py sdist bdist_wheel"
                withCredentials([usernamePassword(credentialsId: 'admin', passwordVariable: 'password', usernameVariable: 'user')]) {
                       sh "twine upload -u ${user} -p ${password} --repository-url http://10.6.13.38:8081/repository/pypi-dataengineering/ dist/*"
                }

            }
        }
        stage('Test install package'){
            steps{
             sh "pip3 install edo-ds-datalake-dags --force-reinstall --extra-index-url http://10.6.13.38:8081/repository/pypi-dataengineering/simple -v --trusted-host 10.6.13.38"
            }
        }
        stage('update pypi package'){
            steps{
             sh "gcloud composer environments update ${COMPOSER_NAME} --location ${LOCATION} --update-pypi-package edo-ds-datalake-dags"
            }
        }
    }
}

