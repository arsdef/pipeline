

# Documentación del Pipeline de GitLab CI

Este documento describe el pipeline de GitLab CI para la construcción, escaneo y despliegue de una aplicación.
El pipeline de GitLab CI se compone de varias etapas interrelacionadas, diseñadas para automatizar de principio a fin el ciclo de integración y despliegue.
La integración con Nexus y Buildpacks permite minimizar la complejidad de algunos pasos del proceso, mientras que el uso de Trivy como herramienta de análisis de vulnerabilidades añade un nivel extra de seguridad al pipeline. Este pipeline se encuentra diseñado para ser modular, escalable y adaptable a distintas configuraciones y requisitos de diferentes proyectos.

## 1. Visión General

El pipeline automatiza el proceso de desarrollo, desde la clonación de repositorios hasta la actualización de etiquetas de imágenes y el escaneo de seguridad.

## 2. Etapas del Pipeline

El pipeline se compone de las siguientes etapas, ejecutadas en el orden especificado:

*   **`clone`**: Clonación de los repositorios de la aplicación y del chart de Helm.
*   **`build`**: Construcción de la imagen de la aplicación utilizando Buildpacks.
*   **`scan`**: Escaneo de seguridad de la imagen construida con Trivy.
*   **`update_tag`**: Actualización de la etiqueta de la imagen en el repositorio de Helm.

## 3. Variables de Configuración

Las siguientes variables son utilizadas para configurar el comportamiento del pipeline:

| Variable | Descripción | Ejemplo |
| :---------------------- | :--------------------------------------------------------------------------------------- | :---------------------------------------------------------- |
| `APP_NAME` | Nombre de la aplicación. | `'my-app'` |
| `NEXUS_URL` | URL de la instancia de Nexus. | `'http://nexus.example.com'` |
| `NEXUS_REPO` | Repositorio de Docker en Nexus donde se almacenarán las imágenes. | `'docker-repo'` |
| `BUILD_PACK_URL` | URL del builder de Buildpacks a utilizar. | `'gcr.io/buildpacks/builder'` |
| `SEVERITY_LEVELS` | Niveles de severidad para el escaneo de seguridad (ej. `CRITICAL,HIGH`). | `'CRITICAL,HIGH'` |
| `NEW_IMAGE_TAG` | Nueva etiqueta de la imagen a utilizar después de la construcción. | `'v1.0.1'` |
| `GITLAB_TOKEN` | Token de GitLab (se usa `$CI_JOB_TOKEN` por defecto). | `$CI_JOB_TOKEN` |
| `NEXUS_TOKEN` | Token para la autenticación en Nexus. | `$NEXUS_TOKEN` |
| `GITLAB_REPO_APP` | URL del repositorio de la aplicación en GitLab. | `'https://gitlab.com/your-group/your-app-repo.git'` |
| `GITLAB_REPO_HELM` | URL del repositorio del chart de Helm en GitLab. | `'https://gitlab.com/your-group/your-helm-chart-repo.git'` |
| `APP_DIRECTORY` | Directorio local donde se clonará el repositorio de la aplicación. | `'your-app-directory'` |
| `HELM_DIRECTORY` | Directorio local donde se clonará el repositorio del chart de Helm. | `'your-helm-chart-directory'` |
| `GIT_BRANCH_APP` | Rama del repositorio de la aplicación a utilizar. | `'main'` |
| `GIT_BRANCH_HELM` | Rama del repositorio del chart de Helm a utilizar. | `'main'` |
| `MAVEN_SETTINGS_PATH` | Ruta al archivo de configuración de Maven (no usado en los pasos actuales, pero declarado). | `".m2/settings.xml"` |
| `NEXUS_REPO_NPM` | Repositorio de NPM en Nexus (no usado en los pasos actuales, pero declarado). | `"npm-hosted"` |

## 4. Descripción de los Trabajos (Jobs)

### 4.1. `clone_application`

*   **Etapa:** `clone`
*   **Imagen:** `alpine/git:latest`
*   **Descripción:** Clona el repositorio de la aplicación especificado por `GITLAB_REPO_APP` en el directorio `APP_DIRECTORY` y en la rama `GIT_BRANCH_APP`.
*   **Restricciones:** Se ejecuta solo en la rama definida por `GIT_BRANCH_APP`.

### 4.2. `clone_helm_chart`

*   **Etapa:** `clone`
*   **Imagen:** `alpine/git:latest`
*   **Descripción:** Clona el repositorio del chart de Helm especificado por `GITLAB_REPO_HELM` en el directorio `HELM_DIRECTORY` y en la rama `GIT_BRANCH_HELM`.
*   **Restricciones:** Se ejecuta solo en la rama definida por `GIT_BRANCH_HELM`.

### 4.3. `update_tag`

*   **Etapa:** `update_tag`
*   **Imagen:** `alpine/git:latest`
*   **Descripción:** Actualiza la etiqueta de la imagen en el archivo `values.yaml` dentro del directorio del chart de Helm. Luego, realiza un `git add`, `git commit` y `git push` para guardar los cambios en el repositorio de Helm.
    *   Instala `sed` para realizar la edición del archivo.
    *   La autenticación para el `git push` se realiza usando el `GITLAB_TOKEN`.
*   **Restricciones:** Se ejecuta solo en la rama definida por `GIT_BRANCH_HELM`.

### 4.4. `build`

*   **Etapa:** `build`
*   **Imagen:** `pbuildpacksio/pack:latest`
*   **Servicios:** `docker:dind` (Docker in Docker)
*   **Descripción:** Construye la imagen de la aplicación utilizando Paketo Buildpacks. La imagen resultante se etiqueta con `NEW_IMAGE_TAG` y se sube al repositorio de Nexus especificado.
    *   Se autentica en Nexus utilizando el `NEXUS_TOKEN`.
*   **Restricciones:** Se ejecuta solo en la rama definida por `GIT_BRANCH_APP`.

### 4.5. `scan`

*   **Etapa:** `scan`
*   **Imagen:** `aquasec/trivy:latest`
*   **Descripción:** Escanea la imagen de la aplicación construida previamente en Nexus utilizando Trivy para identificar vulnerabilidades. El escaneo se limita a los niveles de severidad definidos en `SEVERITY_LEVELS`.
   
plantilla de helm https://github.com/arsdef/plantilla-helm

# Requisitos y Configuración Previa
Antes de implementar el pipeline, es necesario asegurarse de contar con ciertos requisitos y configuraciones previas. Esto incluye, pero no se limita a:

## Credenciales y Accesos:

Acceso a GitLab y permisos para configurar pipelines en el repositorio.
Credenciales válidas para Nexus, de modo que el proceso de subida de artefactos se realice sin interrupciones.
Configuración de variables de entorno en GitLab CI, tales como APP_NAME, NEXUS_URL, BUILD_PACKS_CONFIG, y otros parámetros necesarios para la interacción con las herramientas externas.
Herramientas y Dependencias Instaladas:

## Buildpacks:
Configurados correctamente en el entorno para transformar el código fuente en una imagen Docker.
## Trivy:
Instalado y configurado para ejecutar escaneos y generar reportes de vulnerabilidad.
## Docker:
Disponible en los runners de GitLab para la construcción y gestión de imágenes.
## GitLab Runner:
Configurado en el entorno de ejecución, preferiblemente en un ambiente de contenedores o máquinas virtuales para aprovechar la paralelización de tareas.

# Integración con Nexus, Buildpacks y Herramientas Externas
La integración con herramientas externas es un componente esencial en el pipeline. A continuación, se describen las especificidades y configuraciones necesarias para cada una de ellas:

## Integración con Nexus
Nexus se utiliza como repositorio central de artefactos, donde se almacenan las imágenes Docker construidas. La integración incluye:

### Configuración de la URL de Nexus:
La variable NEXUS_URL debe definir la dirección del servidor Nexus.
### Autenticación:
Es necesario definir las credenciales de acceso en GitLab CI, ya sea mediante variables secretas o archivos de configuración seguros.
Empuje de Imágenes:
Durante la etapa de build, se utiliza el comando docker push para enviar la imagen etiquetada a Nexus.
Integración con Buildpacks
Buildpacks facilita la construcción de imágenes a partir del código fuente sin necesidad de escribir un Dockerfile. Esto permite:

## Automatización y Consistencia:
Se utiliza la herramienta pack para crear imágenes de manera estandarizada.
Configuración del Builder:
Se debe especificar el builder a utilizar, por ejemplo, heroku/buildpacks:20 u otro compatible según las necesidades del proyecto.
Integración con Trivy
Trivy es fundamental para garantizar la seguridad del pipeline. La integración abarca:

## Ejecución del Escáner:
Se utiliza el comando trivy image para analizar la imagen.
Definición de Umbrales:
Se configuran parámetros para que el proceso falle si se detectan vulnerabilidades con niveles CRITICAL o HIGH.
### Salida y Reportes:
El reporte generado puede almacenarse para auditorías o revisiones posteriores.

Todas estas integraciones se configuran de manera que el pipeline mantenga una alta confiabilidad y cumpla con los estándares de seguridad y calidad. Cada herramienta externa se invoca desde el archivo YAML del pipeline, utilizando variables de entorno previamente definidas para mantener la integridad y seguridad de la información.

---
