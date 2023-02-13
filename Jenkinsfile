 pipeline{
        agent any
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
                export AWS_ACCESS_KEY_ID=AKIA36GKNQJI2Y5PIOVC && export AWS_SECRET_ACCESS_KEY=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
                aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/e5k4j6y8
                docker tag project_image:${BUILD_NUMBER} public.ecr.aws/e5k4j6y8/test-ecr:${BUILD_NUMBER}
                docker push public.ecr.aws/e5k4j6y8/test-ecr:${BUILD_NUMBER}
                """
            }
        }
      stage('user_input'){
          steps{
              input("please approve the build")
                 script{
                    sh "echo this is a"
                   }
                }
              }
        stage("Deploy container in server"){
            steps{
            sshagent(['deploy_app']) {
             
              sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.150 docker pull public.ecr.aws/e5k4j6y8/test-ecr:\${BUILD_NUMBER}"
              sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.150 docker rm -f container2 || true"
              sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.150 docker run -d -p 8090:80 --name container2 public.ecr.aws/e5k4j6y8/test-ecr:\${BUILD_NUMBER}"
    
}
    }
        }
        }   // stages closing
         
         post{

 success{
 emailext to: 'govardhanr992@gmail.com',
          subject: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER}.",
          body: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER}.",
          replyTo: 'govardhanr992@gmail.com'
 }
        } // pipeline closing
