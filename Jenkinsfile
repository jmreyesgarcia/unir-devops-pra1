pipeline {
    agent any

    environment {
        JMETER_BIN = 'jmeter.sh'
    }

    stages {
        // NUEVA ETAPA: Requisito de identificación del agente (Reto 2)
        stage('Identificación del Agente') {
            steps {
                sh 'whoami'
                sh 'hostname'
                sh 'python3 --version'
            }
        }

        stage('Análisis Estático (Flake8)') {
            steps {
                sh 'flake8 --exit-zero --format=pylint --output-file=flake8.log app'
                recordIssues tools: [pyLint(pattern: 'flake8.log')],
                             qualityGates: [[threshold: 8, type: 'TOTAL']]
            }
        }

        stage('Seguridad (Bandit)') {
            steps {
                // Bandit genera la salida para el log de consola
                sh 'bandit -r app -f txt || true'
            }
        }

        stage('Tests Unitarios y Cobertura') {
            steps {
                sh 'python3 -m pytest --junitxml=results.xml test/unit'
                sh 'python3 -m coverage run -m pytest test/unit'
                sh 'python3 -m coverage xml'

                junit 'results.xml'
                recordCoverage tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']]
            }
        }

        stage('Performance (JMeter)') {
            steps {
                script {
                    // Lanzamos la API guardando el PID para asegurar su cierre
                    sh 'nohup python3 app/api.py > flask.log 2>&1 & echo $! > api.pid'
                    
                    sleep 5
                    
                    // Ejecución de JMeter usando la variable de entorno
                    sh "${JMETER_BIN} -n -t test_plan.jmx -l results.jtl"
                    
                    // Limpieza obligatoria del proceso
                    sh 'kill $(cat api.pid) || true'
                }
                perfReport sourceDataFiles: 'results.jtl'
            }
        }
    }
}
