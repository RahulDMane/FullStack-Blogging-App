pipeline { 
    agent any
        tools {maven 'maven3' jdk 'jdk17' }
    
    environment { SCANNER_HOME = tool 'sonar-scanner' }

    stages {
        stage('Git Checkout') { steps {git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/RahulDMane/FullStack-Blogging-App.git'}}

        stage('Compile') { steps {sh "mvn compile"}  }

        stage('Test') { steps {sh "mvn test" }  }

        stage('Trivy FS Scan') { steps {"trivy fs --format table -o fs.html ."  } }

        stage('SonarQube Analysis') {steps { withSonarQubeEnv('sonar-server') {
                    sh '''
                    "$SCANNER_HOME/bin/sonar-scanner" \
                        -Dsonar.projectName=Blogging-app \
                        -Dsonar.projectKey=Blogging-app \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes
                    '''  } } }
        stage('Build') { steps { sh "mvn package" }    }

        stage('Publish Artifacts') {steps {withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', traceability: true)
                    {  sh "mvn deploy"}
                                    }     }
        stage('Docker Build & Tag') {steps { 
                 withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') 
                 { sh "docker build -t rahulmane23/bloggingapp:latest ."}
                                    }      }
        stage('Trivy Image Scan') {  steps {
                sh "trivy image --format table -o image.html rahulmane23/bloggingapp:latest"
                                   }        }
        stage('Docker Push Image') { steps {
                withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker')
                { sh "docker push rahulmane23/bloggingapp:latest"     }
                                    }        }
    }
}
