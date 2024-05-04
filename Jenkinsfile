pipeline {
    agent any

    environment {
        STASH_CALCULADORA = 'CALCULADORA'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'agentes', url: 'https://github.com/jdcr-dev/unir-helloworld'                
            }
        }
        
        stage('Build') {            
            steps {                
                stash name: STASH_CALCULADORA, includes: '**'
            }
        }
        
        stage('Test'){
            parallel {
                stage('Unit') {
                    agent { label 'linux-agent-1' }
                    steps {
                        unstash STASH_CALCULADORA

                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                whoami
                                hostname
                                
                                export PYTHONPATH=.
                                pytest --junitxml=result-unit.xml test/unit
                            '''                            
                        }                        
                    }
                }

                stage('Rest'){
                    agent { label 'linux-agent-2' }
                    steps {
                        unstash STASH_CALCULADORA

                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {                    
                            sh '''
                                whoami
                                hostname

                                export FLASK_APP=app/api.py
                                flask run &                                           
                                                        
                                export PYTHONPATH=.
                                pytest --junitxml=result-rest.xml test/rest
                            '''                            
                        }
                    }
                }
            }
        }
        
        
        stage('Results'){            
            parallel {
                stage('Unit') {
                    agent { label 'linux-agent-1' }
                    steps {
                        junit 'result*.xml'
                        echo 'FINISH'
                    }
                }

                stage('Rest') {
                    agent { label 'linux-agent-2' }
                    steps {
                        junit 'result*.xml'
                        echo 'FINISH'
                    }
                }
            }
        }
    }
}
