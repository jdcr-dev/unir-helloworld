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
                                
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        
        
        stage('Results'){
            // when {
            //     expression {
            //         return currentBuild.result != 'FAILURE'
            //     }
            // }
            steps {
                junit 'result*.xml'
                echo 'FINISH'
            }
        }
    }
}
