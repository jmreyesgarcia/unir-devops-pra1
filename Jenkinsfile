pipeline {
    agent any

    environment {
        // Asegúrate de que esta ruta sea correcta en tu servidor de Jenkins
        JMETER_BIN = '/opt/apache-jmeter-5.5/bin/jmeter.sh'
    }

    stages {
        stage('Análisis Estático (Flake8)') {
            steps {
                // Genera log para el plugin Warnings Next Gen
                sh 'flake8 --exit-zero --format=pylint --output-file=flake8.log app'
                recordIssues(tools: [flake8(pattern: 'flake8.log')], unstableTotalAll: 8, failedTotalAll: 10)
            }
        }

        stage('Seguridad (Bandit)') {
            steps {
                sh 'bandit -r app -f json -o bandit.json || true'
                recordIssues(tools: [bandit(pattern: 'bandit.json')], unstableTotalAll: 2, failedTotalAll: 4)
            }
        }

        stage('Tests Unitarios y Cobertura') {
            steps {
                sh 'coverage run -m pytest --junitxml=results.xml test/unit'
                sh 'coverage xml'
                junit 'results.xml'
                // Umbrales CP1.2: Line 85-95% Unstable, Branch 80-90% Unstable
                publishCoverage adapters: [coberturaAdapter('coverage.xml')], 
                    globalThresholds: [[thresholdTarget: 'Line', unstableThreshold: 95.0, unhealthyThreshold: 85.0],
                                     [thresholdTarget: 'Branch', unstableThreshold: 90.0, unhealthyThreshold: 80.0]]
            }
        }

        stage('Performance (JMeter)') {
            steps {
                script {
                    // 1. Arrancar la API en segundo plano
                    sh 'nohup python3 app/api.py > flask.log 2>&1 &'
                    sh 'echo $! > api.pid'
                    sleep 5 // Esperar a que Flask esté listo
                    
                    // 2. Ejecutar JMeter en modo consola (-n)
                    sh "${JMETER_BIN} -n -t test_plan.jmx -l results.jtl"
                    
                    // 3. Matar el proceso de la API
                    sh 'kill $(cat api.pid)'
                }
                perfReport sourceDataFiles: 'results.jtl'
            }
        }
    }
}
