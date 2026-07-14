# local_deployment
SonarQube utiliza Elasticsearch por debajo, lo que exige que el parámetro de mapeo de memoria virtual del sistema operativo anfitrión (vm.max_map_count) sea de al menos 262144.
sudo sysctl -w vm.max_map_count=262144