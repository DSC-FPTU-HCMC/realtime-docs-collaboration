pipeline {
	agent any

	environment {
    GOOGLE_PROJECT_ID = 'dsc-fptu-hcmc-orientation'
  }

  tools {
    nodejs 'NodeJSTool'
  }

	stages {
		stage('Init') {
			steps {
        script {
          if (fileExists('node_modules')) {
            sh 'rm -rf node_modules'
          }

          if (fileExists('rm package-lock.json')) {
            sh 'rm package-lock.json'
          }
        }
        sh '''
          echo "PATH = ${PATH}"
          node -v
          npm -v
          python3 -v
          npm cache clean --force
          npm install
          echo "Init success.."
        '''
			}
		}

		stage('Build') {
			steps {
				sh '''
          echo "BUILD NUMBER = $BUILD_NUMBER"
          ./node_modules/.bin/ng build --aot --prod
          echo "Build Success.."
        '''
			}
			// post {  
      //   always {
      //     notifyThroughEmail('Build-stage')
      //   }
      // }
    }

		stage('Deploy') {
			steps {
        withCredentials([file(credentialsId: 'google-application-credentials-secret-file', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            export DIRECTORY="/var/jenkins_home/GoogleCloudSDK/google-cloud-sdk/bin"
            if [ ! -d "$DIRECTORY" ]; then
              echo "Download Google Cloud SDK tools"
              cd /var/jenkins_home
              wget https://dl.google.com/dl/cloudsdk/release/google-cloud-sdk.zip -O google-cloud-sdk.zip
              unzip -o google-cloud-sdk.zip -d ./GoogleCloudSDK/
              ./GoogleCloudSDK/google-cloud-sdk/install.sh
            fi
            export PATH=/var/jenkins_home/GoogleCloudSDK/google-cloud-sdk/bin:$PATH
            which python3
            export CLOUDSDK_PYTHON=$(which python3)
            echo "PATH: $PATH"
            gcloud --version
            gcloud --quiet components update

            echo "GCP credentails: $GOOGLE_APPLICATION_CREDENTIALS"
            gcloud config set project $GOOGLE_PROJECT_ID
            gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS
            gcloud config list
            
            cp app.yaml dist/app.yaml
            cd dist
            gcloud app deploy --version=v01
            echo "Deployed to Google Compute Engine"
          '''
        }
      }	
      post{
        always{
          println "Result : ${currentBuild.result}"
          // notifyThroughEmail('Deploy-stage')
				}
			}
		}
	}
}

// def notifyThroughEmail(String stage= "Default stage"){
//   emailext  (
//     body:"""
//       Adtech-Service - Build # $BUILD_NUMBER - $currentBuild.currentResult:
    

//       Check console output at $BUILD_URL to view the results.
//     """,
//     compressLog: true,
//     attachLog: true,
//     replyTo: '-----@---.com, -----@----.com',
//     recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//     subject: "Build Notification Jenkins - Project : Project-service - Job: $JOB_NAME Build # $BUILD_NUMBER ${currentBuild.currentResult}",
//     to: '-----@---.com, -----@----.com'
//   )
// }