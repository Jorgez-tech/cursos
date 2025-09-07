# 10. Seguridad en el Ciclo de Vida del Software

## 🛡️ Introducción a DevSecOps

DevSecOps integra prácticas de seguridad en cada fase del ciclo de desarrollo, desde la planificación hasta el deployment y monitoreo en producción.

### Principios de Shift-Left Security

- **Detección temprana**: Identificar vulnerabilidades en fases iniciales
- **Automatización**: Integrar checks de seguridad en CI/CD
- **Cultura de seguridad**: Todos los roles son responsables de la seguridad
- **Feedback continuo**: Retroalimentación rápida sobre issues de seguridad

## 🔒 Seguridad en el Código

### Static Application Security Testing (SAST)

#### SonarQube

```yaml
# sonar-project.properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0
sonar.sources=src
sonar.tests=tests
sonar.language=javascript
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

```yaml
# GitHub Actions - SonarQube
name: SonarQube Analysis
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
        
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests with coverage
      run: npm run test:coverage
      
    - name: SonarQube Scan
      uses: sonarqube-quality-gate-action@master
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

#### ESLint Security Rules

```json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "plugin:security/recommended"
  ],
  "plugins": ["security"],
  "rules": {
    "security/detect-object-injection": "error",
    "security/detect-non-literal-regexp": "error",
    "security/detect-unsafe-regex": "error",
    "security/detect-buffer-noassert": "error",
    "security/detect-child-process": "error",
    "security/detect-disable-mustache-escape": "error",
    "security/detect-eval-with-expression": "error",
    "security/detect-no-csrf-before-method-override": "error",
    "security/detect-non-literal-fs-filename": "error",
    "security/detect-non-literal-require": "error",
    "security/detect-possible-timing-attacks": "error",
    "security/detect-pseudoRandomBytes": "error"
  }
}
```

### Dynamic Application Security Testing (DAST)

#### OWASP ZAP Integration

```yaml
# GitHub Actions - OWASP ZAP
name: Security Testing
on:
  push:
    branches: [ main ]

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3
      
    - name: ZAP Scan
      uses: zaproxy/action-full-scan@v0.4.0
      with:
        target: 'https://www.example.com'
        rules_file_name: '.zap/rules.tsv'
        cmd_options: '-a'
        
    - name: Upload ZAP Report
      uses: actions/upload-artifact@v3
      with:
        name: zap-report
        path: report_html.html
```

## 🔐 Gestión de Secretos

### Usando HashiCorp Vault

```bash
# Instalar Vault
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install vault

# Inicializar Vault
vault server -dev
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="your-root-token"

# Almacenar secretos
vault kv put secret/myapp/config \
    database_url="postgresql://user:password@localhost:5432/mydb" \
    api_key="super-secret-api-key"

# Leer secretos
vault kv get secret/myapp/config
```

#### Integración con Aplicaciones

```javascript
// vault-client.js
const vault = require('node-vault')({
  apiVersion: 'v1',
  endpoint: process.env.VAULT_ADDR,
  token: process.env.VAULT_TOKEN
});

async function getSecrets() {
  try {
    const result = await vault.read('secret/data/myapp/config');
    return result.data.data;
  } catch (error) {
    console.error('Error reading secrets:', error);
    throw error;
  }
}

module.exports = { getSecrets };
```

### AWS Secrets Manager

```yaml
# CloudFormation - Secrets Manager
Resources:
  DatabaseSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Name: MyApp/Database
      Description: Database credentials for MyApp
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: 'password'
        PasswordLength: 32
        ExcludeCharacters: '"@/\'
        
  DatabaseSecretAttachment:
    Type: AWS::SecretsManager::SecretTargetAttachment
    Properties:
      SecretId: !Ref DatabaseSecret
      TargetId: !Ref DatabaseInstance
      TargetType: AWS::RDS::DBInstance
```

```python
# Python - Usando AWS Secrets Manager
import boto3
import json

def get_secret(secret_name, region_name="us-east-1"):
    session = boto3.session.Session()
    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )
    
    try:
        get_secret_value_response = client.get_secret_value(
            SecretId=secret_name
        )
        secret = get_secret_value_response['SecretString']
        return json.loads(secret)
    except Exception as e:
        print(f"Error retrieving secret: {e}")
        raise e

# Uso
db_credentials = get_secret("MyApp/Database")
print(f"Database URL: {db_credentials['username']}")
```

## 🛡️ Seguridad de Contenedores

### Dockerfile Security Best Practices

```dockerfile
# ❌ Inseguro
FROM ubuntu
RUN apt-get update && apt-get install -y python3
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
USER root
CMD ["python3", "app.py"]

# ✅ Seguro
FROM python:3.11-slim-bullseye

# Crear usuario no-root
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Instalar dependencias del sistema como root
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Establecer directorio de trabajo
WORKDIR /app

# Copiar y instalar dependencias Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código de aplicación
COPY --chown=appuser:appuser . .

# Cambiar a usuario no-root
USER appuser

# Exponer puerto (información)
EXPOSE 8000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Comando de inicio
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

### Container Image Scanning

```yaml
# GitHub Actions - Trivy Scan
name: Container Security Scan
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Build image
      run: docker build -t myapp:${{ github.sha }} .
      
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'myapp:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
        
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
```

### Runtime Security with Falco

```yaml
# falco-rules.yaml
- rule: Suspicious Network Activity
  desc: Detect suspicious network connections
  condition: >
    inbound_outbound and
    fd.type=ipv4 and
    (fd.ip != "127.0.0.1" and fd.ip != "::1") and
    not proc.name in (known_network_tools)
  output: >
    Suspicious network activity (user=%user.name command=%proc.cmdline 
    connection=%fd.name)
  priority: WARNING

- rule: Unexpected File Access
  desc: Detect access to sensitive files
  condition: >
    open_read and
    fd.name in (/etc/passwd, /etc/shadow, /etc/sudoers) and
    not proc.name in (authorized_processes)
  output: >
    Sensitive file access (user=%user.name command=%proc.cmdline file=%fd.name)
  priority: CRITICAL
```

## 🔍 Security Scanning en CI/CD

### Comprehensive Security Pipeline

```yaml
# .github/workflows/security.yml
name: Security Pipeline
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
        
    # 1. Dependency Scanning
    - name: Install dependencies
      run: npm ci
      
    - name: Audit dependencies
      run: npm audit --audit-level=high
      
    - name: Check for known vulnerabilities
      run: npx audit-ci --high
      
    # 2. Secret Scanning
    - name: TruffleHog Secret Scan
      uses: trufflesecurity/trufflehog@main
      with:
        path: ./
        base: main
        head: HEAD
        
    # 3. Static Code Analysis
    - name: CodeQL Analysis
      uses: github/codeql-action/init@v2
      with:
        languages: javascript
        
    - name: Autobuild
      uses: github/codeql-action/autobuild@v2
      
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
      
    # 4. License Compliance
    - name: License Check
      run: npx license-checker --onlyAllow 'MIT;Apache-2.0;BSD-3-Clause;ISC'
      
    # 5. Container Security (if applicable)
    - name: Build Docker image
      run: docker build -t security-test:${{ github.sha }} .
      
    - name: Container scan
      run: |
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          -v $HOME/Library/Caches:/root/.cache/ \
          aquasec/trivy image security-test:${{ github.sha }}
          
  infrastructure-security:
    runs-on: ubuntu-latest
    if: contains(github.event.head_commit.modified, 'infrastructure/')
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      
    - name: Terraform Security Scan
      run: |
        curl -L -o tfsec https://github.com/aquasecurity/tfsec/releases/latest/download/tfsec-linux-amd64
        chmod +x tfsec
        ./tfsec infrastructure/ --soft-fail
        
    - name: Checkov Scan
      run: |
        pip install checkov
        checkov -d infrastructure/ --framework terraform
```

## 🏛️ Compliance y Governance

### GDPR Compliance Checklist

```javascript
// gdpr-compliance.js
class GDPRCompliance {
  constructor() {
    this.dataProcessor = new DataProcessor();
    this.consentManager = new ConsentManager();
    this.auditLogger = new AuditLogger();
  }
  
  // Right to be informed
  async collectConsent(userId, dataTypes, purpose) {
    const consent = await this.consentManager.requestConsent({
      userId,
      dataTypes,
      purpose,
      timestamp: new Date(),
      lawfulBasis: 'consent'
    });
    
    this.auditLogger.log('consent_collected', {
      userId,
      consentId: consent.id,
      dataTypes,
      purpose
    });
    
    return consent;
  }
  
  // Right of access
  async getPersonalData(userId) {
    const data = await this.dataProcessor.getUserData(userId);
    
    this.auditLogger.log('data_access_request', {
      userId,
      timestamp: new Date(),
      dataRetrieved: Object.keys(data)
    });
    
    return data;
  }
  
  // Right to rectification
  async updatePersonalData(userId, updates) {
    const oldData = await this.dataProcessor.getUserData(userId);
    const newData = await this.dataProcessor.updateUserData(userId, updates);
    
    this.auditLogger.log('data_rectification', {
      userId,
      oldData,
      newData,
      timestamp: new Date()
    });
    
    return newData;
  }
  
  // Right to erasure
  async deletePersonalData(userId) {
    const deletedData = await this.dataProcessor.deleteUserData(userId);
    
    this.auditLogger.log('data_erasure', {
      userId,
      deletedData,
      timestamp: new Date()
    });
    
    return { success: true, deletedData };
  }
}
```

### SOC 2 Controls Implementation

```yaml
# monitoring-controls.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: soc2-controls
data:
  access-control.yaml: |
    # CC6.1 - Logical and Physical Access Controls
    policies:
      - name: multi-factor-authentication
        required: true
        enforcement: strict
      - name: role-based-access
        required: true
        enforcement: strict
      - name: access-review
        schedule: monthly
        
  system-monitoring.yaml: |
    # CC7.1 - System Monitoring
    monitoring:
      - metric: system_availability
        threshold: 99.9%
        alert: true
      - metric: response_time
        threshold: 2s
        alert: true
      - metric: error_rate
        threshold: 0.1%
        alert: true
        
  change-management.yaml: |
    # CC8.1 - Change Management
    procedures:
      - name: code-review
        required: true
        minimum_reviewers: 2
      - name: testing
        required: true
        coverage_threshold: 80%
      - name: deployment-approval
        required: true
        approvers: ["security-team", "operations-team"]
```

## 🚨 Incident Response

### Security Incident Playbook

```yaml
# incident-response-playbook.yml
name: Security Incident Response
trigger:
  - security_alert
  - vulnerability_detection
  - data_breach_suspected

phases:
  preparation:
    actions:
      - ensure_incident_response_team_ready
      - verify_communication_channels
      - check_backup_systems
      
  identification:
    actions:
      - assess_incident_scope
      - classify_incident_severity
      - notify_stakeholders
    timeline: 1 hour
    
  containment:
    actions:
      - isolate_affected_systems
      - preserve_evidence
      - implement_temporary_fixes
    timeline: 4 hours
    
  eradication:
    actions:
      - remove_threat_vectors
      - patch_vulnerabilities
      - update_security_controls
    timeline: 24 hours
    
  recovery:
    actions:
      - restore_systems_from_clean_backups
      - implement_additional_monitoring
      - validate_system_integrity
    timeline: 48 hours
    
  lessons_learned:
    actions:
      - conduct_post_incident_review
      - update_procedures
      - provide_training
    timeline: 1 week
```

```python
# incident_response.py
import logging
import json
from datetime import datetime
from enum import Enum

class IncidentSeverity(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4

class SecurityIncident:
    def __init__(self, incident_type, description, severity):
        self.id = self.generate_incident_id()
        self.incident_type = incident_type
        self.description = description
        self.severity = severity
        self.created_at = datetime.now()
        self.status = "OPEN"
        self.actions = []
        
    def generate_incident_id(self):
        return f"SEC-{datetime.now().strftime('%Y%m%d')}-{hash(datetime.now()) % 10000:04d}"
        
    def add_action(self, action, user):
        self.actions.append({
            "action": action,
            "user": user,
            "timestamp": datetime.now(),
            "status": "COMPLETED"
        })
        
    def escalate(self, new_severity):
        old_severity = self.severity
        self.severity = new_severity
        self.add_action(f"Escalated from {old_severity} to {new_severity}", "system")
        
        if new_severity == IncidentSeverity.CRITICAL:
            self.notify_executives()
            
    def notify_executives(self):
        # Enviar notificación a ejecutivos
        notification = {
            "incident_id": self.id,
            "severity": self.severity.name,
            "description": self.description,
            "created_at": self.created_at.isoformat()
        }
        logging.critical(f"EXECUTIVE NOTIFICATION: {json.dumps(notification)}")
        
    def close_incident(self, resolution_notes):
        self.status = "CLOSED"
        self.closed_at = datetime.now()
        self.resolution_notes = resolution_notes
        self.add_action("Incident closed", "incident_response_team")
```

## ⚙️ Ejercicios Prácticos

1. **SAST Integration**: Configurar SonarQube en pipeline CI/CD
2. **Secret Management**: Implementar Vault para gestión de secretos
3. **Container Security**: Configurar scanning automático de imágenes
4. **Incident Response**: Simular y responder a incidente de seguridad
5. **Compliance Audit**: Implementar controles para SOC 2 o ISO 27001

## 🎯 Checklist de Seguridad

### Development Phase
- ✅ Code review con enfoque en seguridad
- ✅ SAST integrado en IDE y CI/CD
- ✅ Dependency scanning automatizado
- ✅ Secret scanning en commits
- ✅ Security training para developers

### Testing Phase
- ✅ DAST en entornos de testing
- ✅ Penetration testing regular
- ✅ Security regression testing
- ✅ Infrastructure security testing

### Deployment Phase
- ✅ Container image scanning
- ✅ Infrastructure as Code security
- ✅ Zero-trust network architecture
- ✅ Monitoring y alerting configurado

### Operations Phase
- ✅ Security incident response plan
- ✅ Regular security assessments
- ✅ Compliance monitoring
- ✅ Backup y disaster recovery tested

## 🔗 Recursos Adicionales

- [OWASP Top 10](https://owasp.org/Top10/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/controls/)
- [DevSecOps.org](https://www.devsecops.org/)
- [Security Code Warrior](https://www.securecodewarrior.com/)

---

**Anterior**: [09. Cloud](./09-cloud.md) | **Inicio**: [README](../README.md)