pipeline {
    agent any

    environment {
        JMETER_BIN = 'jmeter.sh' 
    }

    stages {
        stage('Análisis Estático (Flake8)') {
            steps {
                sh 'flake8 --exit-zero --format=pylint --output-file=flake8.log app'
                recordIssues tools: [pyLint(pattern: 'flake8.log')], 
                             qualityGates: [[threshold: 8, type: 'TOTAL']]
            }
        }

        stage('Seguridad (Bandit)') {
            steps {
                sh 'bandit -r app -f txt || true' 
            }
        }

        stage('Tests Unitarios y Cobertura') {
            steps {
                // Ejecutamos los tests y generamos el xml de cobertura
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
                    sh 'nohup python3 app/api.py > flask.log 2>&1 &'
                    sh 'echo $! > api.pid'
                    sleep 5 
                    sh "${JMETER_BIN} -n -t test_plan.jmx -l results.jtl"
                    sh 'kill $(cat api.pid) || true'
                }
                perfReport sourceDataFiles: 'results.jtl'
            }
        }
    }
}
