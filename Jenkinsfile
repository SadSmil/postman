pipeline{
    agent any
    parameters{
        booleanParam(name:'ALLURE', defaultValue: false, description: 'generation de rapport allure')
        //choice(name: 'browser', choices: ['firefox','chromium','webkit'], description: 'Choisissez le choix du navigateur')   
    }
    stages{
        stage('global stage'){
             agent {
                docker { 
                    image 'node:latest'
                    args '-u=root --entrypoint='
                }  
            }
            stages{
                stage('install deps'){
                    steps{
                        sh 'npm install'
                        sh 'npm install newman'
                        sh 'npm install newman-reporter-allure'
                            
                    }
                }

                stage('clean allure results'){
                    
                    steps{
                        sh '''
                    echo "Suppression du cache Allure..."
                    rm -rf allure-results
                    mkdir -p allure-results
                    echo "Dossier allure-results nettoyé avec succès"
                    '''
                    }
                }
        
               stage('run newman tests') {
                    steps {
                        script {
                            def baseCmd = "npx newman run collection1.json"

                            if (params.ALLURE) {
                                sh "${baseCmd} -r cli,allure --reporter-allure-export allure-results"
                                echo "je suis dans le if de allure"
                                stash name: 'allure-results', includes: 'allure-results/*'
                            } else {
                                sh "${baseCmd} -r htmlextra,cli --reporter-htmlextra-export newman-report.html"
                            }
                        }
                    }
                }
            }
        }
    }
    post{
        success{
            script{
                if(params.ALLURE){
                    unstash 'allure-results'
                    archiveArtifacts 'allure-results/*'
                    allure includeProperties: false,
                           jdk: '',
                           results: [[path: 'allure-results/']]
                }
            }
        }
    }
}