pipeline {
    agent any

    environment {
        IMAGE_NAME = "4ddocker/lab1:${env.BUILD_NUMBER}"
        IMAGE_LATEST = "4ddocker/lab1:latest"
        LOCAL_DATA_PATH = "C:\\DopEdu\\ML_ITMO\\DevOps_BigFiles"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📦 Клонирование репозитория из GitHub...'
                checkout scm
                echo '✅ Код успешно получен'
            }
        }

        stage('Copy Large Files') {
            steps {
                echo '📁 Копирование больших файлов (данные и модель) из локальной папки...'
                bat """
                    if not exist "data" mkdir data
                    if exist "${LOCAL_DATA_PATH}\\data\\data_graduates_university_specialty_124_v20250709.csv" (
                        copy "${LOCAL_DATA_PATH}\\data\\data_graduates_university_specialty_124_v20250709.csv" data\\
                    ) else (
                        echo "CSV file not found, skipping"
                    )
                    if not exist "models" mkdir models
                    if exist "${LOCAL_DATA_PATH}\\models\\*.pkl" (
                        copy "${LOCAL_DATA_PATH}\\models\\*.pkl" models\\
                    ) else (
                        echo "Model files not found, skipping"
                    )
                """
                echo '✅ Большие файлы скопированы'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🏗️ Сборка Docker образа...'
                bat "docker build -t ${IMAGE_NAME} ."
                bat "docker tag ${IMAGE_NAME} ${IMAGE_LATEST}"
                echo '✅ Образ собран'
            }
        }

        stage('Test Container') {
            steps {
                echo '🧪 Запуск тестового контейнера...'
                bat """
                    docker run -d --name test-container-${env.BUILD_NUMBER} -p 8888:8000 ${IMAGE_NAME}
                    timeout /t 10 /nobreak > nul
                    curl.exe -f http://localhost:8888/health
                    docker stop test-container-${env.BUILD_NUMBER}
                    docker rm test-container-${env.BUILD_NUMBER}
                """
                echo '✅ Контейнер успешно протестирован'
            }
        }
    }

    post {
        always {
            bat "docker rmi ${IMAGE_NAME} ${IMAGE_LATEST} || true"
        }
        success {
            echo '🎉 Pipeline успешно выполнен!'
        }
        failure {
            echo '❌ Pipeline завершился с ошибкой. Проверьте логи выше.'
        }
    }
}