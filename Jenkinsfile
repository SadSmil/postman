pipeline {
    agent any
    parameters {
        booleanParam(name: 'ALLURE', defaultValue: true, description: 'Génération du rapport Allure (si décoché, utilise htmlextra)')
    }
    stages {
        stage('Global Stage') {
            agent {
                docker { 
                    image 'node:20-alpine' // Utiliser une version LTS stable est plus safe que 'latest'
                    args '-u=root --entrypoint='
                }  
            }
            stages {
                stage('Install Deps') {
                    steps {
                        // Utilisation de npm ci ou installation locale propre
                        sh 'npm init -y' // Sécurité si pas de package.json existant
                        sh 'npm install newman newman-reporter-allure newman-reporter-htmlextra'
                    }
                }

                stage('Clean Allure Results') {
                    steps {
                        sh '''
                        echo "Suppression du cache Allure..."
                        rm -rf allure-results newman-report.html
                        mkdir -p allure-results
                        '''
                    }
                }
        
                stage('Run Newman Tests') {
                    steps {
                        script {
                            def baseCmd = "npx newman run collection1.json"
                            
                            // block try/catch ou gestion de statut pour éviter que le pipeline crash direct en cas de test KO
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
                                    // Le stash doit être fait ICI, à l'intérieur de l'agent Docker, juste après le run
                                    stash name: 'allure-results-stash', includes: 'allure-results/**'
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always { // 'always' pour générer le rapport même si les tests échouent
            script {
                if (params.ALLURE) {
                    try {
                        unstash 'allure-results-stash'
                        
                        // Déclenchement du plugin Allure Jenkins
                        allure includeProperties: false,
                               jdk: '',
                               results: [[path: 'allure-results']]
                    } catch (Exception e) {
                        echo "Impossible de générer le rapport Allure (le stash est peut-être vide) : ${e.message}"
                    }
                } else {
                    // Si htmlextra a été choisi, on archive le fichier HTML
                    archiveArtifacts artifacts: 'newman-report.html', allowEmptyArchive: true
                }
            }
        }
    }
}