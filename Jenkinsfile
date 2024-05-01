pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git 'https://github.com/jdcr-dev/unir-helloworld'
                // bat 'git clone <URL>'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Eyyy, esto es Python. No hay que compilar nada!!!'
                echo WORKSPACE
                bat 'dir'
            }
        }
        
        stage('Test'){
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
        
                stage('Rest'){
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set FLASK_APP=app/api.py
                                start flask run
                            
                                start java -jar c:\\Python39\\Lib\\site-packages\\wiremock\\server\\wiremock-standalone-2.35.1.jar --port 9090 --verbose --root-dir test\\wiremock
                                
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        
        
        stage('Results'){
            steps {
                junit 'result*.xml'
                echo 'FINISH'
            }
        }
    }
}
