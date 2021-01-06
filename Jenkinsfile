def analyzer_svc = 'analyzer'
def publisher_svc = 'publisher'
def notifier_svc = 'notifier'
def streamer_svc = 'streamer'
def svc_port = 5000
def skipRemainingStages = false
def app_name = 'app'
def main_branch = 'master'
def host = 'localhost'
def registry_name = 'registry'
def cred = 'cred'
def check_script_name = '/var/lib/jenkins/scripts/check_app.sh'
def test_image = 'hello-world'
pipeline{
    agent{
        label main_branch
    }
    stages{
        stage("Checkout"){
            steps{
               script {
                   try {
                        dir(app_name){
                        git branch: main_branch,
                        credentialsId: cred,
                        url: 'https://github.com/LidorAbo/' + app_name + '.git'
                    }
                   }
                   catch(exception) {
                      skipRemainingStages = true
                   }
                    
               }
            }
        }
        stage("Sanity check"){
            when{
                expression {
                    !skipRemainingStages
                }
            }
            steps{
                script{
                    try{
                        dir(app_name){
                            sh """
                             for svc in ${analyzer_svc} ${notifier_svc} ${publisher_svc} ${streamer_svc}; do
                                ${check_script_name} \$svc ${svc_port}
                            done
                            """
                        } 
                    }
                    catch (exception) {
                        skipRemainingStages = true
                    }
                        
                }
            }
        }
       
        stage("Test local private registry"){
            when{
                expression {
                    !skipRemainingStages
                }
            }
            steps{
                script{
                    try{   
                        sh """
                        docker > /dev/null 2>&1 pull ${test_image}
                        docker > /dev/null 2>&1 tag ${test_image} ${host}:${svc_port}/${test_image}
                        docker > /dev/null 2>&1 push ${host}:${svc_port}/${test_image}
                        docker > /dev/null 2>&1 rmi -f ${test_image} ${host}:${svc_port}/${test_image}
                        docker > /dev/null 2>&1 run ${host}:${svc_port}/${test_image}
                        """
                           
                    }
                    catch(exception) {
                       skipRemainingStages = true   
                    }
                }
            }
        }
        stage('Build services containers and push to local registry'){
            when{
                expression {
                    !skipRemainingStages
                }
            }
            steps{
                script{
                    try {
                        dir(app_name){
                                 sh """
                       for svc in ${analyzer_svc} ${notifier_svc} ${publisher_svc} ${streamer_svc}; do
                            docker build -f \$svc/Dockerfile \$svc -t \$svc
                            docker tag \$svc ${host}:${svc_port}/\$svc
                            docker push ${host}:${svc_port}/\$svc
                       done
                       """
                      
                   }
                    }
                    catch (exception) {
                        skipRemainingStages = true
                    }
                   
                }
            }
        }
        stage('Triggering CD job'){
            when{
                expression {
                    !skipRemainingStages
                }
            }
            steps {
                script{
                        build job: 'CD', wait: false, parameters: [
                     string(name: "svc_port", value: "${svc_port}"),
                     string(name: "analyzer_svc", value: "${analyzer_svc}"),
                     string(name: "notifier_svc", value: "${notifier_svc}"),
                     string(name: "publisher_svc", value: "${publisher_svc}"),
                     string(name: "streamer_svc", value: "${streamer_svc}"),

                     ]     
                }
            }
        }
    }
    post{
        always{
            sh """
            docker 2> /dev/null images | grep -E "${analyzer_svc}|${notifier_svc}|${publisher_svc}|${streamer_svc}|${test_image}" | awk '{print \$3}' | xargs docker rmi -f || true 
            docker system prune -f 
            """
        }
    }
}

