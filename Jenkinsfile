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
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                if (params.ALLURE) {
                                    sh "${baseCmd} -r cli,allure --reporter-allure-export allure-results"
                                } else {
                                    sh "${baseCmd} -r htmlextra,cli --reporter-htmlextra-export newman-report.html"
                                }
                            }

                            // CORRECTION ICI : On cible directement le dossier complet sans le pattern bloquant
                            if (params.ALLURE) {
                                stash name: 'allure-results-stash', includes: 'allure-results/'
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
                    // On nettoie d'abord l'espace de l'agent Jenkins pour éviter les conflits
                    sh 'rm -rf allure-results' 
                    
                    unstash 'allure-results-stash'
                    
                    // Appel du plugin Allure
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