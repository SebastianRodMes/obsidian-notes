# ============================================
# PREPARACIÓN DEL PROYECTO
# ============================================

# 1. Verificar estructura del proyecto
ls -la
# Debe mostrar: Dockerfile, requirements.txt, src/

# 2. Crear cuenta en Docker Hub (si no tienes)
# Ir a: https://hub.docker.com/

# ============================================
# CONSTRUCCIÓN DE LA IMAGEN
# ============================================

# 3. Construir la imagen Docker localmente
docker build -t mi-api-rest .

# Alternativa con tag específico
docker build -t mi-api-rest:v1.0 .

# 4. Verificar que la imagen se creó correctamente
docker images

# 5. Probar la imagen localmente (opcional pero recomendado)
docker run -d -p 5001:5001 --name test-api mi-api-rest
docker logs test-api
# Si funciona, detener y eliminar el contenedor de prueba
docker stop test-api
docker rm test-api

# ============================================
# PREPARACIÓN PARA DOCKER HUB
# ============================================

# 6. Iniciar sesión en Docker Hub
docker login
# Te pedirá username y password de Docker Hub

# 7. Etiquetar la imagen con tu username de Docker Hub
docker tag mi-api-rest tu-username/mi-api-rest:latest
docker tag mi-api-rest tu-username/mi-api-rest:v1.0

# Ejemplo específico (reemplaza "sebas" con tu username real):
docker tag mi-api-rest sebas/mi-api-rest:latest
docker tag mi-api-rest sebas/mi-api-rest:v1.0

# 8. Verificar las imágenes etiquetadas
docker images | grep mi-api-rest

# ============================================
# SUBIDA A DOCKER HUB
# ============================================

# 9. Subir la imagen a Docker Hub
docker push tu-username/mi-api-rest:latest
docker push tu-username/mi-api-rest:v1.0

# Ejemplo específico:
docker push sebas/mi-api-rest:latest
docker push sebas/mi-api-rest:v1.0

# ============================================
# VERIFICACIÓN Y LIMPIEZA
# ============================================

# 10. Verificar que se subió correctamente
# Ir a: https://hub.docker.com/r/tu-username/mi-api-rest

# 11. Probar la descarga desde Docker Hub (opcional)
docker pull tu-username/mi-api-rest:latest

# 12. Limpiar imágenes locales (opcional)
docker rmi mi-api-rest
docker rmi tu-username/mi-api-rest:latest
docker rmi tu-username/mi-api-rest:v1.0

# ============================================
# COMANDOS ÚTILES ADICIONALES
# ============================================

# Ver el historial de la imagen
docker history tu-username/mi-api-rest:latest

# Ver información detallada de la imagen
docker inspect tu-username/mi-api-rest:latest

# Cerrar sesión de Docker Hub
docker logout

# ============================================
# ACTUALIZACIÓN DE VERSIONES (FUTURAS)
# ============================================

# Para futuras actualizaciones:
# 1. Modificar código
# 2. Reconstruir imagen:
docker build -t tu-username/mi-api-rest:v1.1 .
# 3. Actualizar latest:
docker tag tu-username/mi-api-rest:v1.1 tu-username/mi-api-rest:latest
# 4. Subir nuevas versiones:
docker push tu-username/mi-api-rest:v1.1
docker push tu-username/mi-api-rest:latest