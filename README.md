# local_deployment
1. **SonarQube** utiliza Elasticsearch por debajo, lo que exige que el parámetro de mapeo de memoria virtual del sistema operativo anfitrión (vm.max_map_count) sea de al menos 262144.
```bash
sudo sysctl -w vm.max_map_count=262144
```
2. **Levantar el entorno**

```bash
docker compose up -d
```
3. **Verificar los logs**
```bash
docker compose logs -f sonarqube
```
Cuando veas que dice SonarQube is operational, ya estará listo.

4. **Obtener clave de Jenkins**
```bash
docker exec jenkins_local cat /var/jenkins_home/secrets/initialAdminPassword
```
Una vez que todo esté corriendo, podrás acceder a Jenkins en http://localhost:8080 y a SonarQube en http://localhost:9000 (usuario y contraseña por defecto: admin / admin).

# Seguridad
**Paso 1: Generar un almacén de claves (Keystore)**
Tanto Jenkins como SonarQube (al estar basados en Java) prefieren trabajar con un archivo de almacén de claves seguro en lugar de llaves de texto plano .crt o .key.

```bash
mkdir -p security
keytool -genkeypair -alias keystore_local -keyalg RSA -keysize 2048 \
  -validity 365 -keystore security/keystore.p12 -storetype PKCS12 \
  -storepass mi_clave_secreta -dname "CN=localhost, OU=DevOps, O=Local, L=SantaClara, S=VillaClara, C=CU"
```
**Nota:** Guarda bien la contraseña que elijas.

**Paso 2: Aplicar y Validar**
```bash
docker compose down
docker compose up -d
```
# Jenkins
1. Configure GitHub, SonarQube and Kubernetes tokens.
2. Install the "SonarQube Scanner for Jenkins" plugin.
3. Add SonarQube servers under Configuration > System.

**Install kubectl in Jenkins container**
For exec container as root:
```bash
docker exec -u 0 -it jenkins bash
```
Inside the container, run the fallowing commands to download and install kubectl the official kubectl binary
```bash
# 1. Download the kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
# 2. Add executable permission and move it to the system path.
chmod +x kubectl
mv kubectl /usr/local/bin/
# 3. Verify the installation
kubectl version --client
# 4. Exit the container
exit
```

# SonarQube
1. Generate token for Jenkins in My Account>Security. 
2. Create a webhook in Administration>Configuration>Webhooks
(http://jenkins:8080/sonarqube-webhook/)

