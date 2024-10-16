pipeline {
  agent any
  stages {
    stage('build') {
      agent {
        docker {
          image 'maven:3.9.6-eclipse-temurin-17'
        }

      }
      steps {
        echo 'compile maven app'
        sh 'mvn compile'
      }
    }

    stage('test') {
      agent {
        docker {
          image 'maven:3.9.6-eclipse-temurin-17'
        }

      }
      steps {
        echo 'test maven app'
        sh 'mvn clean test'
      }
    }

    stage('package') {
      parallel {
        stage('package') {
          agent {
            docker {
              image 'maven:3.9.6-eclipse-temurin-17'
            }

          }
          steps {
            echo 'package maven app'
            sh 'mvn package -DskipTests'
            archiveArtifacts 'target/*.jar'
          }
        }

        stage('Docker B&P') {
          agent any
          steps {
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                // Fetch the full Git commit hash and truncate it to the first 7 characters
                def commitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
                // Build the Docker image with the short commit hash as tag
                // replace xxxxxx with your docker hub username/org
                def dockerImage = docker.build("disha4091/sysfoo:${commitHash}", "./")
                // Push the image tagged with the commit hash
                dockerImage.push()
                // Push additional standard tags if needed
                dockerImage.push("latest")
                dockerImage.push("dev")
              }
            }

          }
        }

        stage('Deploy to Dev') {
          when {
                 beforeAgent true
                 branch  'main'
               }
    
          agent any
    
          steps {
            echo 'Deploying to Dev Environment with Docker Compose'
            sh 'docker-compose up -d'
          }
      }

      }
    }

  }
  tools {
    maven 'Maven 3.6.3'
  }
}
