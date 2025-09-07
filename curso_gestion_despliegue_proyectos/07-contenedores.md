# 07. Contenedores y Virtualización

## 🐳 Introducción a Contenedores

Los contenedores permiten empaquetar aplicaciones y sus dependencias en entornos aislados, facilitando la portabilidad y el despliegue.

### Historia y evolución

- **2001**: Virtualización tradicional (VMware, VirtualBox)
- **2013**: Docker populariza los contenedores
- **Actualidad**: Orquestación (Kubernetes), microservicios

## 🏗️ Conceptos Clave

- Imagen y contenedor
- Dockerfile y docker-compose
- Orquestación (Kubernetes)
- Volúmenes y redes

## ⚡ Ejemplo: Dockerfile básico para app Node.js

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

## 📝 Ejercicio Práctico

1. Crea un Dockerfile para una app Python o Node.js.
2. Despliega un contenedor y accede a la aplicación localmente.

## 🎯 Buenas Prácticas

- Mantén las imágenes ligeras y seguras
- Versiona los archivos Dockerfile
- Usa orquestadores para proyectos grandes

---

**Siguiente**: [08. Infraestructura como Código (IaC)](./08-iac.md)
