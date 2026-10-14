pipeline{
    agent{
        node{
            label 'prod'
        }
    }
    stages{
        stage("checkout code"){
            steps{
                git 'https://github.com/Sravani-Meduri/python-code-library-app.git'
            }
        }
         stage('Build') {
            steps {
                sh '''
                    python3 --version
                    python3 -m pip --version
                    python3 -m pip install -r requirements.txt
                    python3 -m compileall -q .
                '''
    }
         stage('SonarQube Analysis') {
             steps{
                 script{
                  def scannerHome = tool 'mySonar';
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                 }
             }
         }
         stage("Quality Gates"){
             steps{
                 waitForQualityGate abortPipeline: true, credentialsId: 'sonarid'
             }
         }
         stage("Build images"){
             steps{
                 sh '''
                    docker build -t frontned-image:v1 .
                    docker build -t auth-image:v1 auth/
                    docker build -t book-image:v1 book/
                    docker build -t borrow-image:v1 borrow/
                    docker build -t db-image:v1 database/
                 '''
             }
         }
         stage("Scan the images"){
             steps{
                 sh '''
                    trivy image auth-image:v1
                    trivy image book-image:v1
                    trivy image borrow-image:v1
                    trivy image frontned-image:v1
                    trivy image db-image:v1
                 '''
             }
         }
         stage("Push to registry"){
             steps{
                 script{
                     withDockerRegistry(credentialsId: 'HubID') {
                        sh 'docker tag auth-image:v1 sravanif5/flm-oct:pythonappauth'
                        sh 'docker tag book-image:v1 sravanif5/flm-oct:pythonappbook'
                        sh 'docker tag borrow-image:v1 sravanif5/flm-oct:pythonappborrow'
                        sh 'docker tag frontned-image:v1 sravanif5/flm-oct:pythonappfrontned'
                        sh 'docker tag db-image:v1 sravanif5/flm-oct:db'
                        sh 'docker push sravanif5/flm-oct:pythonappauth'
                        sh 'docker push sravanif5/flm-oct:pythonappbook'
                        sh 'docker push sravanif5/flm-oct:pythonappborrow'
                        sh 'docker push sravanif5/flm-oct:pythonappfrontned'
                        sh 'docker push sravanif5/flm-oct:db'
                     }
                     
                 }
             }
         }
         stage("Deploy using Stack"){
             steps{
                 sh 'docker stack deploy myapplication --compose-file=compose.yml'
             }
         }
    }
}
