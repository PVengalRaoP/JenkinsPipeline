pipeline {
    agent any
    tools {
        gradle 'GradleBuild' 
    }
    environment{
        registry="vengi014/pipeline"
        registryCredential = 'DockerHub_id'
        dockerImage=''
    }
    stages{
            stage('Code Checkout')
            {
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/PVengalRaoP/JenkinsTest.git']]])
            }
            }
            
            
            stage('Build') {
            steps {
                //sh 'gradle --version'
                sh 'gradle clean build'
                //sh 'ls -l'
                //sh 'pwd'
            }
            }
            
            
             stage('Jmeter Report generation')
            {
                steps{
                sh '/root/apache-jmeter-5.4.3/bin/jmeter.sh -Jjmeter.save.saveservice.output_format=xml -n -t /root/apache-jmeter-5.4.3/bin/JavaViewResultsInTable.jmx -l report.jtl'
            }
                
            }
           
            
            stage('SonarQube Analysis')
            {steps{
            withSonarQubeEnv(installationName: 'SonarQube') {
                 // some block
                 sh "gradle sonarqube"
            }}
            }
            
            
            stage("Quality gate") {
            steps 
            {
                waitForQualityGate abortPipeline: true
            }
          
        }
        stage('building our image')
        {
            steps{
                script
                {
                    dockerImage= docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Clair Scan')
        {
            steps{
                sh 'docker run --net=web --rm --name=scanner --link=clair:clair -v "/tmp/logers/:/var/log/" -v /var/run/docker.sock:/var/run/docker.sock objectiflibre/clair-scanner --clair="http://clair:6060" --ip="scanner" -r "/var/lib/jenkins/workspace/PipelineProject/httpd_log.json" -t Low $registry:$BUILD_NUMBER'
            }
        }
        stage('Push image into DockerHub')
        {
            steps{
                script{
                    docker.withRegistry( '', registryCredential)
                    { dockerImage.push() }
                }
            }
        }
         /*
        stage ("Prompt for input") {
          steps {
            script {
                env.Var = input message: 'Please enter "yes" if you are satisfied with vulnerabilities result to push DockerImage into DockerHub'
                    }
                }
        }
        stage("Pushing or PushNot"){
         steps
         {
             script {
                    if (env.Var == 'yes') {
                        echo 'I will push'
                    } else {
                        echo 'I will not push'
                    }
         }
         }
        }*/
        
    }
        
        post { 
        always { 
            
            junit 'app/build/test-results/test/TEST-*.xml'
            
            perfReport filterRegex: '', modePerformancePerTestCase: true, modeThroughput: true, showTrendGraphs: true, sourceDataFiles: 'report.jtl'
        }
    }
    
    }