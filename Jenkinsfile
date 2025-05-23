pipeline {
    agent any

    environment {
        NODE_RED_HOST = "http://localhost:1880"
        RABBITMQ_API = "http://localhost:15672/api/queues"
        LOGSTASH_CONF_DIR = "/etc/logstash/conf.d"
        KIBANA_HOST = "http://localhost:5601"
        ELASTIC_USER = "elastic"
        ELASTIC_PASS = "PlnLz35OqHQ1UAOLqo8b"
    }

    stages {
        stage('Deploy Node-RED Flow') {
            steps {
                sh '''
                curl -X POST $NODE_RED_HOST/flows \
                     -H "Content-Type: application/json" \
                     -d @node_red_flow.json
                '''
            }
        }

        stage('Create RabbitMQ Queue') {
            steps {
                sh '''
                curl -u guest:guest -H "Content-Type: application/json" \
                     -X PUT "$RABBITMQ_API/%2f/my-queue" \
                     -d '{"durable":true}'
                '''
            }
        }

        stage('Deploy Logstash Config') {
            steps {
                sh '''
                sudo cp logstash.conf $LOGSTASH_CONF_DIR/
                sudo systemctl restart logstash
                '''
            }
        }

        stage('Register Kibana Template') {
            steps {
                sh '''
                curl -X POST -u $ELASTIC_USER:$ELASTIC_PASS \
                     -H "kbn-xsrf: true" \
                     -H "Content-Type: application/json" \
                     -d @kibana_template.json \
                     "$KIBANA_HOST/api/saved_objects/index-pattern/my-index-*?overwrite=true"
                '''
            }
        }

        stage('Send Sample Data (via Node-RED)') {
            steps {
                sh '''
                curl -X POST "$NODE_RED_HOST/sample-inject"
                '''
            }
        }

        stage('Create Kibana Index Pattern') {
            steps {
                sh '''
                curl -X POST -u $ELASTIC_USER:$ELASTIC_PASS \
                     -H "kbn-xsrf: true" \
                     -H "Content-Type: application/json" \
                     -d '{"attributes":{"title":"my-index-*","timeFieldName":"@timestamp"}}' \
                     "$KIBANA_HOST/api/saved_objects/index-pattern/my-index-*"
                '''
            }
        }
    }
}
