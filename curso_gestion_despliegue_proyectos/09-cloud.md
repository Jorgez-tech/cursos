# 09. Cloud y Opciones de Hosting

## ☁️ Introducción al Cloud Computing

El cloud computing es la entrega de servicios informáticos a través de Internet, incluyendo servidores, almacenamiento, bases de datos, redes, software, análisis e inteligencia.

### Modelos de Servicio

#### Infrastructure as a Service (IaaS)
- **Control total** sobre la infraestructura virtual
- **Ejemplos**: AWS EC2, Azure Virtual Machines, Google Compute Engine
- **Ideal para**: Migración lift-and-shift, control granular

#### Platform as a Service (PaaS)
- **Plataforma gestionada** para desarrollo y deployment
- **Ejemplos**: Heroku, AWS Elastic Beanstalk, Azure App Service
- **Ideal para**: Desarrollo rápido, focus en código

#### Software as a Service (SaaS)
- **Aplicaciones completas** disponibles bajo demanda
- **Ejemplos**: Gmail, Salesforce, Microsoft 365
- **Ideal para**: Usuarios finales, sin gestión técnica

### Modelos de Deployment

#### Cloud Público
- Recursos compartidos y gestionados por proveedores
- **Ventajas**: Costo bajo, escalabilidad, sin mantenimiento
- **Desventajas**: Menos control, posibles concerns de seguridad

#### Cloud Privado
- Infraestructura dedicada para una organización
- **Ventajas**: Control total, seguridad, personalización
- **Desventajas**: Mayor costo, complejidad de gestión

#### Cloud Híbrido
- Combinación de cloud público y privado
- **Ventajas**: Flexibilidad, optimización de costos
- **Desventajas**: Complejidad de integración

## 🚀 Principales Proveedores de Cloud

### Amazon Web Services (AWS)

#### Servicios Principales

```bash
# AWS CLI - Comandos básicos
aws configure
aws s3 ls
aws ec2 describe-instances
aws cloudformation deploy --template-file template.yaml --stack-name my-stack
```

#### EC2 (Elastic Compute Cloud)
```bash
# Lanzar instancia EC2
aws ec2 run-instances \
    --image-id ami-0c02fb55956c7d316 \
    --instance-type t2.micro \
    --key-name my-key-pair \
    --security-group-ids sg-12345678 \
    --subnet-id subnet-12345678
```

#### S3 (Simple Storage Service)
```bash
# Crear bucket
aws s3 mb s3://my-unique-bucket-name

# Subir archivo
aws s3 cp index.html s3://my-unique-bucket-name/

# Sincronizar directorio
aws s3 sync ./dist s3://my-unique-bucket-name/
```

#### Lambda (Serverless Functions)
```python
# lambda_function.py
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Hello from Lambda!',
            'input': event
        })
    }
```

### Microsoft Azure

#### Servicios Principales

```bash
# Azure CLI - Comandos básicos
az login
az group create --name myResourceGroup --location eastus
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name myUniqueAppName
```

#### App Service
```yaml
# azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'
    
- script: |
    npm install
    npm run build
  displayName: 'npm install and build'
  
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'My Azure Subscription'
    appType: 'webApp'
    appName: 'myUniqueAppName'
    package: '$(System.DefaultWorkingDirectory)/dist'
```

#### Azure Functions
```csharp
// Function.cs
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;

public class HttpTriggerFunction
{
    [Function("HttpTrigger")]
    public async Task<HttpResponseData> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData req)
    {
        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteStringAsync("Hello from Azure Functions!");
        return response;
    }
}
```

### Google Cloud Platform (GCP)

#### Servicios Principales

```bash
# gcloud CLI - Comandos básicos
gcloud auth login
gcloud config set project my-project-id
gcloud app deploy
```

#### App Engine
```yaml
# app.yaml
runtime: nodejs18

env_variables:
  NODE_ENV: production
  
automatic_scaling:
  min_instances: 1
  max_instances: 10
```

#### Cloud Functions
```javascript
// index.js
exports.helloWorld = (req, res) => {
  res.send('Hello from Google Cloud Functions!');
};
```

## 🏗️ Arquitecturas Cloud Comunes

### Arquitectura de 3 Capas

```mermaid
graph TD
    A[Load Balancer] --> B[Web Tier]
    B --> C[Application Tier]
    C --> D[Database Tier]
    
    B --> B1[Web Server 1]
    B --> B2[Web Server 2]
    
    C --> C1[App Server 1]
    C --> C2[App Server 2]
    
    D --> D1[Primary DB]
    D --> D2[Read Replica]
```

### Microservicios en Cloud

```yaml
# docker-compose.yml para desarrollo
version: '3.8'
services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
      
  api-gateway:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - user-service
      - product-service
      
  user-service:
    build: ./user-service
    environment:
      - DATABASE_URL=postgresql://user:pass@user-db:5432/users
    depends_on:
      - user-db
      
  product-service:
    build: ./product-service
    environment:
      - DATABASE_URL=postgresql://user:pass@product-db:5432/products
    depends_on:
      - product-db
      
  user-db:
    image: postgres:13
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      
  product-db:
    image: postgres:13
    environment:
      POSTGRES_DB: products
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
```

### Serverless Architecture

```yaml
# serverless.yml
service: my-serverless-app

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1

functions:
  api:
    handler: src/handler.api
    events:
      - httpApi:
          path: /{proxy+}
          method: ANY
      
  processQueue:
    handler: src/handler.processQueue
    events:
      - sqs:
          arn: arn:aws:sqs:us-east-1:123456789012:my-queue
          
resources:
  Resources:
    MyQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: my-queue
        
    MyBucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: my-unique-bucket-${self:provider.stage}
```

## 💰 Optimización de Costos

### Estrategias de Cost Management

#### 1. Right-Sizing de Recursos

```bash
# AWS - Analizar utilización de instancias
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
    --start-time 2023-01-01T00:00:00Z \
    --end-time 2023-01-02T00:00:00Z \
    --period 3600 \
    --statistics Average
```

#### 2. Auto Scaling

```json
{
  "AutoScalingGroupName": "my-asg",
  "MinSize": 1,
  "MaxSize": 10,
  "DesiredCapacity": 2,
  "DefaultCooldown": 300,
  "TargetGroupARNs": [
    "arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/73e2d6bc24d8a067"
  ],
  "Tags": [
    {
      "Key": "Environment",
      "Value": "production",
      "PropagateAtLaunch": true
    }
  ]
}
```

#### 3. Reservaciones y Spot Instances

```bash
# AWS - Comprar Reserved Instances
aws ec2 purchase-reserved-instances-offering \
    --reserved-instances-offering-id 649fd0c8-73eb-4d8b-8a46-aa1234567890 \
    --instance-count 1

# Usar Spot Instances
aws ec2 request-spot-instances \
    --spot-price "0.05" \
    --instance-count 1 \
    --type "one-time" \
    --launch-specification file://specification.json
```

### Monitoreo de Costos

```yaml
# CloudWatch Billing Alarm
Resources:
  BillingAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: BillingAlarm
      AlarmDescription: Alarm when AWS bill exceeds $100
      MetricName: EstimatedCharges
      Namespace: AWS/Billing
      Statistic: Maximum
      Period: 86400
      EvaluationPeriods: 1
      Threshold: 100
      ComparisonOperator: GreaterThanThreshold
      Dimensions:
        - Name: Currency
          Value: USD
      AlarmActions:
        - !Ref SNSTopic
```

## 📊 Monitoreo y Observabilidad

### Logging Centralizado

```yaml
# docker-compose.logging.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      
  logstash:
    image: docker.elastic.co/logstash/logstash:7.14.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
      
  kibana:
    image: docker.elastic.co/kibana/kibana:7.14.0
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    depends_on:
      - elasticsearch
```

### Métricas y Alertas

```yaml
# Prometheus configuration
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api-service'
    static_configs:
      - targets: ['api:3000']
    metrics_path: '/metrics'
    
  - job_name: 'database'
    static_configs:
      - targets: ['db-exporter:9104']

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

```yaml
# alert_rules.yml
groups:
  - name: api_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors per second"
          
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }} seconds"
```

## 🔒 Seguridad en Cloud

### Identity and Access Management (IAM)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::my-bucket"
    }
  ]
}
```

### Network Security

```yaml
# Security Groups
Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for web servers
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref BastionSecurityGroup
          
  DatabaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for database
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          SourceSecurityGroupId: !Ref WebSecurityGroup
```

## ⚙️ Ejercicios Prácticos

1. **Deploying a Static Website**: Desplegar sitio estático en S3 + CloudFront
2. **Containerized Application**: Desplegar aplicación en ECS/Fargate
3. **Serverless API**: Crear API REST con Lambda + API Gateway
4. **Database Migration**: Migrar base de datos on-premise a RDS
5. **Monitoring Setup**: Implementar logging y métricas centralizadas

## 🎯 Buenas Prácticas

- ✅ **Multi-AZ Deployment**: Distribuir aplicaciones en múltiples zonas
- ✅ **Auto Scaling**: Configurar escalamiento automático
- ✅ **Backup Strategy**: Implementar backups automatizados
- ✅ **Cost Optimization**: Monitorear y optimizar costos regularmente
- ✅ **Security First**: Aplicar principio de menor privilegio
- ✅ **Disaster Recovery**: Planificar estrategias de recuperación

## 🔗 Recursos Adicionales

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
- [Cloud Native Computing Foundation](https://www.cncf.io/)
- [The Twelve-Factor App](https://12factor.net/)

---

**Anterior**: [08. IaC](./08-iac.md) | **Siguiente**: [10. Seguridad](./10-seguridad.md) | **Inicio**: [README](../README.md)