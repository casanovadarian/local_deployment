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