 pipeline{
        agent { node { label "slave1" }}
        environment {
           ACCESS_KEY= "AKIA36GKNQJI2Y5PIOVC"
           SECRET_ACCESS_KEY= "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"
           
         
        options {
           buildDiscarder(logRotator(numToKeepStr: '5'))
           timeout(time: 30, unit: 'MINUTES')
     stages{
        stage("Checkout"){
            steps{
                    git branch: 'main', credentialsId: 'Github_cred', url: 'https://github.com/govardhan992/project1.git'

            }
        }
        stage("Build Docker Image"){
            steps{

             sh """
             docker build -t project_image:${BUILD_NUMBER} .
             """
            }
        }
        stage("Push to ecr registry"){
            steps{
                sh """
                export AWS_ACCESS_KEY_ID=${ACCESS_KEY} && export AWS_SECRET_ACCESS_KEY=${SECRET_ACCESS_KEY}
                aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/e5k4j6y8
                docker tag project_image:${BUILD_NUMBER} public.ecr.aws/e5k4j6y8/test-ecr:${BUILD_NUMBER}
                docker push public.ecr.aws/e5k4j6y8/test-ecr:${BUILD_NUMBER}
                """
            }
        }
      
        stage("Deploy container in server"){
            steps{
             script {

                 def USER_INPUT = input(
                    message: 'User input required - Do you want to proceed?',
                    parameters: [
                            [$class: 'ChoiceParameterDefinition',
                             choices: ['no','yes'].join('\n'),
                             name: 'input',
                             description: 'Menu - select box option']
                    ])
                    echo "The answer is: ${USER_INPUT}"
                    if( "${USER_INPUT}" == "yes"){
                     
            sshagent(['deploy_app']) {
             
              sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.150 docker pull public.ecr.aws/e5k4j6y8/test-ecr:\${BUILD_NUMBER}"
              sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.150 docker rm -f webserver || true"
              sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.150 docker run -d -p 8090:80 --name webserver public.ecr.aws/e5k4j6y8/test-ecr:\${BUILD_NUMBER}"
    
}
    }  else {
                   echo "Skipping deployment"
        }
             }
            }
        }
      
        }   // stages closing
         
         post{

 success{
 emailext to: 'govardhanr992@gmail.com',
          subject: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          body: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          replyTo: 'govardhanr992@gmail.com'
 }
          failure{
 emailext to: 'govardhanr9922@gmail.com',
          subject: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          body: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          replyTo: 'govardhanr992@gmail.com'
 }
  always {
          cleanWs()
          }
 
}
        } // pipeline closing
