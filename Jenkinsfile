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
        
        stage('Start Flask') {
            steps {
                bat '''
                    set FLASK_APP=app/api.py
                    start flask run
                '''
            }
        }

        stage('Wait for Flask') {
            steps {
                script {
                    def retries = 0
                    def maxRetries = 3
                    def urlToCheck = 'http://localhost:5000'

                    // Esperamos que la aplicación este lista o que alcance la espera máxima que consideremos
                    while (retries < maxRetries) {           
                        def curlCommand = "curl -s -o NUL -w %%{http_code} ${urlToCheck}"                                                  
                        def httpResponse = bat(script: curlCommand, returnStdout: true).trim()                        

                        if (httpResponse.tokenize().last() == '200') {
                            echo "Info: Flask Arrancado"
                            break
                        } else {
                            echo "Warning: Esperando que Flask arranque"
                            sleep time: 5, unit: 'SECONDS'
                            retries++
                        }
                    }

                    if (retries == maxRetries) {
                        error "Timeout: La aplicación no arrancó pasados ${maxRetries * 5} segundos"
                    }
                }
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
