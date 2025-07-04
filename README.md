# CI/CD Pipeline con GitLab CI para Proyectos Java y Node.js

Este pipeline de GitLab CI está diseñado para detectar automáticamente el tipo de proyecto (Java con Maven/Gradle o Node.js con npm) y ejecutar solo las etapas necesarias para ese lenguaje. Además, realiza análisis de calidad de código, empaquetado con Paketo Buildpacks, subida a Nexus y escaneo de vulnerabilidades con Trivy.

---

## 🔍 Detección del tipo de proyecto

El pipeline utiliza `rules: exists` para detectar automáticamente si el repositorio contiene:

- `pom.xml` → Proyecto Java con Maven
- `build.gradle` → Proyecto Java con Gradle
- `package.json` → Proyecto Node.js con npm

Según el archivo detectado, se ejecutan los jobs correspondientes.

---

## 🧩 Etapas del Pipeline

1. **configure_nexus**: Configura credenciales para Nexus (Docker, Maven, npm).
2. **build_and_test**:
   - `java_build_and_test`: Compila y prueba proyectos Maven.
   - `gradle_build_and_test`: Compila y prueba proyectos Gradle.
   - `npm_build_and_test`: Instala dependencias y ejecuta pruebas npm.
3. **analyze_code**:
   - `sonar_java_analyze`: Análisis de código Java con SonarQube.
   - `sonar_npm_analyze`: Análisis de código JavaScript con SonarQube.
4. **package_image**:
   - `package_with_buildpacks`: Empaqueta la aplicación con Paketo y publica la imagen Docker en Nexus.
5. **scan_image**:
   - `scan_docker_image_trivy`: Escanea la imagen Docker con Trivy para detectar vulnerabilidades.
6. **deploy_app**:
   - `deploy_to_dev`: Despliega la imagen en un entorno de desarrollo (simulado).

---

## ☁️ Publicación en Nexus

- Las imágenes Docker se publican en el registro Nexus usando `pack build --publish`.
- Las credenciales se configuran automáticamente en `.configure_nexus_base`.
- También se generan archivos `settings.xml` para Maven y `.npmrc` para npm con autenticación.

---

## 🔐 Escaneo de Seguridad

- Se utiliza Trivy para escanear la imagen Docker generada.
- Se configuran severidades y tiempo de espera mediante variables (`TRIVY_SEVERITY`, `TRIVY_TIMEOUT`, etc.).
- El escaneo puede fallar el pipeline si se encuentran vulnerabilidades críticas.

---

## ✅ Requisitos

- GitLab Runner con soporte Docker.
- Acceso a instancias de Nexus y SonarQube.
- Variables de entorno configuradas: `NEXUS_USERNAME`, `NEXUS_PASSWORD`, `SONAR_TOKEN`, etc.

---

## 📦 Herramientas utilizadas

- **Paketo Buildpacks**: Para empaquetar aplicaciones sin Dockerfile.
- **SonarQube**: Para análisis de calidad de código.
- **Trivy**: Para escaneo de vulnerabilidades en imágenes Docker.
- **Maven / Gradle / npm**: Para compilación y pruebas.

---

Este pipeline es flexible, extensible y adecuado para proyectos híbridos o monorepositorios.
