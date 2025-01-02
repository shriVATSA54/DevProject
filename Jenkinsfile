@Library("Shared") _
pipeline {
    agent  { label 'agentluffy' }
    stages {
        stage('hello'){
            steps{
                script{
                    echo 'Hello World'                    
                }
            }
        }
        stage('Setup Files') {
            steps {
                script {
                    // Check if Dockerfile exists, if not create it
                    if (!fileExists('Dockerfile')) {
                        echo 'Dockerfile does not exist, creating Dockerfile...'
                        writeFile file: 'Dockerfile', text: '''
                        FROM python:3.9

WORKDIR /app/backend

COPY requirements.txt /app/backend
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*


# Install app dependencies
RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

COPY . /app/backend

EXPOSE 8000
#RUN python manage.py migrate
#RUN python manage.py makemigrations
                        
                        '''
                    } else {
                        echo 'Dockerfile already exists, skipping creation.'
                    }

                    // Check if requirements.txt exists (for Python projects), if not create it
                    if (!fileExists('requirements.txt')) {
                        echo 'requirements.txt does not exist, creating requirements.txt...'
                        writeFile file: 'requirements.txt', text: '''
                        asgiref==3.6.0
Django==4.1.5
django-cors-headers==3.13.0
djangorestframework==3.14.0
gunicorn==20.1.0
pytz==2022.7.1
sqlparse==0.4.3
tzdata==2022.7
whitenoise==6.3.0
                        '''
                    } else {
                        echo 'requirements.txt already exists, skipping creation.'
                    }

                    // You can also check and create package.json for Node.js (if needed)
                    if (!fileExists('package.json')) {
                        echo 'package.json does not exist, creating package.json...'
                        writeFile file: 'package.json', text: '''
                        {
                            "name": "notes-app",
                            "version": "1.0.0",
                            "main": "index.js",
                            "dependencies": {
                                "express": "^4.17.1"
                            }
                        }
                        '''
                    } else {
                        echo 'package.json already exists, skipping creation.'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    bat "docker build -t notes-app:latest ."
                }
            }
        }

        // Add more stages for tests, deployment, etc.

        stage('Push to DockerHub') {
            steps {
                echo "Push image to Docker"
                withCredentials([usernamePassword(credentialsId: 'DockerHubCreds', passwordVariable: 'dockerHubPass', usernameVariable: 'dockerHubUser')]) {
                    script {
                        try {
                            // Log into DockerHub
                            bat "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                            // Tag the Docker image
                            bat "docker image tag 90dayofdevops-app:latest ${env.dockerHubUser}/90dayofdevops-app:latest"
                            // Push the Docker image to DockerHub
                            bat "docker push ${env.dockerHubUser}/90dayofdevops-app:latest"
                        } catch (Exception e) {
                            error "Docker push failed: ${e.getMessage()}"
                        }
                    }
                }
            }
        }
    }
}
