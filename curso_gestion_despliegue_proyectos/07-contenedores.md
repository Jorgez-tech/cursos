# 07. Contenedores: Docker y Kubernetes

## 🐳 Introducción a Contenedores

Los contenedores revolucionaron el despliegue de aplicaciones al proporcionar una forma eficiente de empaquetar software y sus dependencias en unidades portables y ligeras que pueden ejecutarse de forma consistente en cualquier entorno.

### Historia y evolución

- **1979**: Chroot jail - Primeros aislamiento de procesos en Unix
- **2001-2008**: Virtualización tradicional (VMware, Xen, VirtualBox)
- **2008**: LXC (Linux Containers) - Primera implementación moderna de contenedores
- **2013**: Docker - Popularización masiva de contenedores
- **2014**: Kubernetes - Orquestación de contenedores a escala
- **Actualidad**: Ecosystem maduro con Docker, Podman, containerd, Kubernetes

### Ventajas de los contenedores

- **Portabilidad**: "Funciona en mi máquina" → "Funciona en todas las máquinas"
- **Eficiencia**: Menor overhead que VMs tradicionales
- **Escalabilidad**: Inicio rápido y gestión dinámica de recursos
- **Consistencia**: Mismo comportamiento en desarrollo, testing y producción
- **Microservicios**: Facilita arquitecturas distribuidas
- **DevOps**: Integración natural con CI/CD

## 🏗️ Conceptos Clave

### Diferencia entre Imagen y Contenedor
- **Imagen**: Template inmutable que contiene la aplicación y dependencias
- **Contenedor**: Instancia ejecutable de una imagen
- **Registry**: Repositorio donde se almacenan las imágenes (Docker Hub, ECR, ACR)
- **Layer**: Cada instrucción en Dockerfile crea una capa inmutable

### Dockerfile vs Docker Compose
- **Dockerfile**: Define cómo construir una imagen individual
- **Docker Compose**: Orquesta múltiples contenedores como aplicación completa

### Contenedores vs Máquinas Virtuales
| Aspecto | Contenedores | VMs Tradicionales |
|---------|-------------|------------------|
| **Overhead** | Bajo (MB) | Alto (GB) |
| **Inicio** | Segundos | Minutos |
| **Aislamiento** | Proceso | Sistema completo |
| **Portabilidad** | Alta | Media |
| **Seguridad** | Compartido kernel | Aislamiento completo |

## ⚡ Docker en la Práctica

### Dockerfile para aplicación Node.js
```dockerfile
# Usar imagen oficial Node.js como base
FROM node:18-alpine AS builder

# Establecer directorio de trabajo
WORKDIR /app

# Copiar archivos de dependencias primero (para cache layer)
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production && npm cache clean --force

# Copiar código fuente
COPY . .

# Construir aplicación (si es necesario)
RUN npm run build

# Etapa de producción más pequeña
FROM node:18-alpine AS production

# Crear usuario no-root por seguridad
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Establecer directorio de trabajo
WORKDIR /app

# Copiar desde etapa builder solo lo necesario
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./package.json

# Cambiar a usuario no-root
USER nextjs

# Exponer puerto
EXPOSE 3000

# Variables de entorno
ENV NODE_ENV=production
ENV PORT=3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Comando por defecto
CMD ["node", "dist/index.js"]
```

### Docker Compose para stack completo
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Base de datos PostgreSQL
  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp_prod
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp_user -d myapp_prod"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Cache Redis
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Aplicación principal
  app:
    build: 
      context: .
      dockerfile: Dockerfile
      target: production
    environment:
      NODE_ENV: production
      DB_HOST: database
      DB_USER: myapp_user
      DB_PASSWORD: ${DB_PASSWORD}
      DB_NAME: myapp_prod
      REDIS_HOST: redis
      REDIS_PASSWORD: ${REDIS_PASSWORD}
    ports:
      - "3000:3000"
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - app-network
    restart: unless-stopped
    volumes:
      - ./logs:/app/logs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/ssl/certs
      - nginx_logs:/var/log/nginx
    depends_on:
      - app
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  nginx_logs:

networks:
  app-network:
    driver: bridge
```

### Comandos Docker esenciales
```bash
# Construcción y gestión de imágenes
docker build -t myapp:latest .
docker build -t myapp:v1.2.0 --target production .
docker images
docker rmi myapp:old-version

# Gestión de contenedores
docker run -d --name myapp -p 3000:3000 myapp:latest
docker ps
docker logs myapp
docker exec -it myapp /bin/sh
docker stop myapp
docker rm myapp

# Docker Compose
docker-compose up -d
docker-compose logs -f app
docker-compose exec app /bin/sh
docker-compose down
docker-compose pull  # Actualizar imágenes

# Gestión de volúmenes
docker volume ls
docker volume inspect postgres_data
docker volume prune  # Limpiar volúmenes no utilizados

# Limpieza del sistema
docker system df     # Ver uso de espacio
docker system prune  # Limpiar recursos no utilizados
docker system prune -a --volumes  # Limpieza completa
```

## ☸️ Introducción a Kubernetes

Kubernetes es una plataforma de orquestación de contenedores que automatiza el despliegue, escalado y gestión de aplicaciones containerizadas a gran escala.

### Componentes principales
- **Pod**: Unidad mínima de despliegue (uno o más contenedores)
- **Deployment**: Gestiona réplicas de pods
- **Service**: Proporciona acceso de red a pods
- **Ingress**: Gestiona acceso externo HTTP/HTTPS
- **ConfigMap/Secret**: Configuración y datos sensibles
- **Namespace**: Aislamiento lógico de recursos

### Ejemplo: Despliegue Kubernetes básico
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:v1.2.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-password
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

## 📝 Ejercicios Prácticos

### Ejercicio 1: Containerizar aplicación completa
1. Crea una aplicación web simple (Node.js/Python/PHP)
2. Escribe Dockerfile optimizado con multi-stage build
3. Crea docker-compose.yml con base de datos y cache
4. Implementa health checks y logging
5. Prueba el stack completo localmente

### Ejercicio 2: Optimización de imágenes
1. Construye imagen básica de tu aplicación
2. Optimiza usando Alpine Linux, multi-stage builds
3. Implementa security scanning (Trivy, Clair)
4. Compara tamaños antes y después
5. Documenta mejoras obtenidas

### Ejercicio 3: Orquestación con Docker Compose
1. Diseña stack con múltiples servicios
2. Configura networking entre contenedores
3. Implementa persistent volumes
4. Agrega monitoring básico (Prometheus/Grafana)
5. Automatiza backups de datos

## 🎯 Buenas Prácticas

### Construcción de imágenes
- **Minimize layers**: Agrupa comandos RUN relacionados
- **Use multi-stage builds**: Reduce tamaño final de imagen
- **Alpine Linux**: Para imágenes base más pequeñas
- **Non-root user**: Ejecuta aplicaciones con usuario limitado
- **.dockerignore**: Excluye archivos innecesarios del build context

### Seguridad
- **Scan vulnerabilities**: Usa herramientas como Trivy, Clair
- **Secrets management**: Nunca incluyas credenciales en imágenes
- **Read-only filesystem**: Monta filesystem como read-only cuando sea posible
- **Resource limits**: Define límites de CPU y memoria
- **Network policies**: Restringe comunicación entre pods

### Monitoreo y logging
- **Health checks**: Implementa endpoints de salud
- **Structured logging**: Usa formato JSON para logs
- **Centralized logging**: ELK stack, Fluentd, Loki
- **Metrics**: Expose métricas para Prometheus
- **Distributed tracing**: Jaeger, Zipkin para microservicios

### Producción
- **Immutable infrastructure**: Nunca modifiques contenedores en running
- **Blue/green deployments**: Zero-downtime deployments
- **Auto-scaling**: Horizontal Pod Autoscaler en Kubernetes
- **Backup strategy**: Automated backups de persistent volumes
- **Disaster recovery**: Plan para recuperación de servicios

## 🔧 Herramientas del Ecosystem

### Desarrollo local
- **Docker Desktop**: GUI para Docker en Windows/macOS
- **Minikube**: Kubernetes local para desarrollo
- **Kind**: Kubernetes en Docker para testing
- **Skaffold**: Desarrollo continuo en Kubernetes

### Registries
- **Docker Hub**: Registry público más popular
- **Amazon ECR**: Elastic Container Registry de AWS
- **Google GCR**: Google Container Registry
- **Azure ACR**: Azure Container Registry
- **Harbor**: Registry on-premises con scanning

### Orquestación alternativa
- **Docker Swarm**: Orquestación nativa de Docker (más simple)
- **Nomad**: Orchestrator de HashiCorp
- **OpenShift**: Kubernetes enterprise de Red Hat
- **Rancher**: Plataforma de gestión de Kubernetes

---

**Siguiente**: [08. Infraestructura como Código (IaC)](./08-iac.md)
