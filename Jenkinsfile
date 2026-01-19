pipeline {
    agent any

    environment {
        // Asegúrate de que esta ruta es la correcta en tu consola Linux
        // Si no estás seguro, ejecuta 'which jmeter' o busca el binario .sh
        JMETER_BIN = 'jmeter.sh'
    }

    stages {
        stage('Análisis Estático (Flake8)') {
            steps {
                // Ejecuta flake8 y genera el log en formato pylint
                sh 'flake8 --exit-zero --format=pylint --output-file=flake8.log app'
                // Registra los problemas. Si hay > 8, el build será UNSTABLE (Amarillo)
                recordIssues tools: [pyLint(pattern: 'flake8.log')], 
                             qualityGates: [[threshold: 8, type: 'TOTAL', severity: 'UNSTABLE']]
            }
        }

        stage('Seguridad (Bandit)') {
            steps {
                // Ejecuta bandit ignorando errores de salida para que el pipeline no se detenga
                sh 'bandit -r app -f json -o bandit.json || true'
                // Registra los problemas de seguridad
                recordIssues tools: [bandit(pattern: 'bandit.json')], 
                             qualityGates: [[threshold: 2, type: 'TOTAL', severity: 'UNSTABLE']]
            }
        }

        stage('Tests Unitarios y Cobertura') {
            steps {
                // Ejecuta tests y genera reporte de cobertura
                sh 'coverage run -m pytest --junitxml=results.xml test/unit'
                sh 'coverage xml'
                
                // Publica resultados de tests unitarios
                junit 'results.xml'
                
                // Publica reporte de cobertura (Plugin Cobertura)
                publishCoverage adapters: [coberturaAdapter('coverage.xml')]
            }
        }

        stage('Performance (JMeter)') {
            steps {
                script {
                    // 1. Levantar la aplicación en segundo plano
                    sh 'nohup python3 app/api.py > flask.log 2>&1 &'
                    sh 'echo $! > api.pid'
                    
                    // Esperar a que el microservicio esté listo
                    sleep 5 
                    
                    // 2. Ejecutar JMeter (5 hilos, 40 peticiones según test_plan.jmx)
                    sh "${JMETER_BIN} -n -t test_plan.jmx -l results.jtl"
                    
                    // 3. Detener la aplicación
                    sh 'kill $(cat api.pid) || true'
                }
                // Publica el reporte de rendimiento
                perfReport sourceDataFiles: 'results.jtl'
            }
        }
    }
}
