pipeline {
    agent any

    environment {
        // Asegúrate de que esta ruta es la correcta en tu Linux
        JMETER_BIN = '/opt/apache-jmeter-5.5/bin/jmeter.sh'
    }

    stages {
        stage('Análisis Estático (Flake8)') {
            steps {
                sh 'flake8 --exit-zero --format=pylint --output-file=flake8.log app'
                recordIssues tools: [flake8(pattern: 'flake8.log')], 
                             qualityGates: [[threshold: 8, type: 'TOTAL', severity: 'UNSTABLE'],
                                           [threshold: 10, type: 'TOTAL', severity: 'ERROR']]
            }
        }

        stage('Seguridad (Bandit)') {
            steps {
                sh 'bandit -r app -f json -o bandit.json || true'
                recordIssues tools: [bandit(pattern: 'bandit.json')], 
                             qualityGates: [[threshold: 2, type: 'TOTAL', severity: 'UNSTABLE'],
                                           [threshold: 4, type: 'TOTAL', severity: 'ERROR']]
            }
        }

        stage('Tests Unitarios y Cobertura') {
            steps {
                sh 'coverage run -m pytest --junitxml=results.xml test/unit'
                sh 'coverage xml'
                junit 'results.xml'
                
                // Sintaxis simplificada para cobertura
                publishCoverage adapters: [coberturaAdapter('coverage.xml')]
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
