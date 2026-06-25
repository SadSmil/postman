pipeline {
    agent any
    parameters {
        booleanParam(name: 'ALLURE', defaultValue: true, description: 'Génération du rapport Allure')
    }
    stages {
        stage('Global Stage') {
            agent {
                docker { 
                    image 'node:20-alpine'
                    args '-u=root --entrypoint='
                }  
            }
            stages {
                stage('Install Newman & Reporters') {
                    steps {
                        sh 'npm install -g newman newman-reporter-allure newman-reporter-htmlextra'
                    }
                }

                stage('Clean Allure Results') {
                    steps {
                        sh '''
                        echo "Suppression du cache..."
                        rm -rf allure-results newman-report.html
                        mkdir -p allure-results
                        '''
                    }
                }
        
                stage('Run Newman Tests') {
                    steps {
                        script {
                            def baseCmd = "newman run collection1.json"
                            
                            // On utilise catchError pour être sûr que le stash s'exécute même si Newman échoue
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                if (params.ALLURE) {
                                    sh "${baseCmd} -r cli,allure --reporter-allure-export allure-results"
                                } else {
                                    sh "${baseCmd} -r htmlextra,cli --reporter-htmlextra-export newman-report.html"
                                }
                            }

                            if (params.ALLURE) {
                                stash name: 'allure-results-stash', includes: 'allure-results/**', allowEmpty: true
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                if (params.ALLURE) {
                    unstash 'allure-results-stash'
                    allure includeProperties: false,
                           jdk: '',
                           results: [[path: 'allure-results']]
                } else {
                    archiveArtifacts artifacts: 'newman-report.html', allowEmptyArchive: true
                }
            }
        }
    }
}