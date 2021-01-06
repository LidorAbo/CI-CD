def orchestrate_tool = 'K8s'
def skipRemainingStages = false
def path_sep = '/'
def script_name = path_sep + 'var' + path_sep + 'lib' + path_sep + 'jenkins' + path_sep +'scripts' + path_sep + 'expose.sh'
pipeline {
    agent{
        label 'master'
    }
    stages{
        stage('Checkout'){
            steps{
               script {
                   try {
                        dir(orchestrate_tool){
                        git branch: 'main',
                        credentialsId: 'cred',
                        url: 'https://github.com/LidorAbo/DevOps-App.git'
                    }
                    
                   }
                   catch(exception) {
                      skipRemainingStages = true
                   }
                    
               }
            }
        }
        stage('Deploy K8s services'){
            when {
                expression {
                    !skipRemainingStages
                }
            }
            steps{
                script{
                    try{
                        dir(orchestrate_tool){
                            sh """
                                kubectl apply -f db-persistent-volume.yaml
                                for svc in ${params.analyzer_svc} ${params.notifier_svc} ${params.publisher_svc} ${params.streamer_svc}; do
                                        kubectl apply -f \$svc-${orchestrate_tool}${path_sep}
                                done
                                
                       """
                        }
                        

                    }
                    catch(exception) {
                        skipRemainingStages = true
                    }
                }
            }
        }
        stage('Expose ports locally'){
            steps{
                     parallel(
               
                        streamer: {
                                 script{
                                     try{
                                        sh "${script_name} ${params.streamer_svc}"
                                     }
                                     catch(exception){
                                         currentBuild.result = 'SUCCESS'
                                     }
                          
                            }
                           
                        },
                        analyzer: {
                            script{
                                try{
                                     sh "${script_name} ${params.analyzer_svc} 5002"
                                }
                                catch(exception){
                                    currentBuild.result = 'SUCCESS'

                                }
                            }
                        },
                        notifier: {
                            script{
                                try{
                                    sh "${script_name} ${params.notifier_svc} 5003"
                                }
                                catch(exception){
                                    currentBuild.result = 'SUCCESS'
                                }
                            }
                        },
                        publisher: {
                        script {
                            try{
                                sh "${script_name} ${params.publisher_svc} 5004"
                            }
                            catch(exception){
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                        }
                    
                     )         
            }
        }
     }
    post{
        always{
            dir('/var/lib/jenkins' + path_sep + 'workspace'){ 
                 cleanWs()
            }
        }
    }
  }