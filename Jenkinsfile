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
                        // Installation globale (-g) pour contourner le problème de nom de dossier de Jenkins
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
                            // On appelle newman directement (sans npx car installé avec -g)
                            def baseCmd = "newman run collection1.json"
                            
                            try {
                                if (params.ALLURE) {
                                    sh "${baseCmd} -r cli,allure --reporter-allure-export allure-results"
                                } else {
                                    sh "${baseCmd} -r htmlextra,cli --reporter-htmlextra-export newman-report.html"
                                }
                            } catch (Exception e) {
                                currentBuild.result = 'UNSTABLE'
                                echo "Certains tests Postman ont échoué, mais on continue pour générer le rapport."
                            } finally {
                                if (params.ALLURE) {
                                    stash name: 'allure-results-stash', includes: 'allure-results/**', allowEmpty: true
                                }
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
                    try {
                        unstash 'allure-results-stash'
                        allure includeProperties: false,
                               jdk: '',
                               results: [[path: 'allure-results']]
                    } catch (Exception e) {
                        echo "Impossible de générer le rapport Allure : ${e.message}"
                    }
                } else {
                    archiveArtifacts artifacts: 'newman-report.html', allowEmptyArchive: true
                }
            }
        }
    }
}