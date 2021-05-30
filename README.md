# Automatización CI/CD Jenkins de Grafana en Infraestructura Existente.

En este repositorio automatizaremos con Jenkins el despliegue de un servidor grafana dentro de una infraestructura de red core de acuerdo  al repositorio  https://github.com/ezequiellladoce/Deploy_Jenkins_in_Created_Infra.git. Utilizaremos:

  - Jenkins server creado en  https://github.com/ezequiellladoce/Deploy_Jenkins_in_Created_Infra.git para ejecutar el pipeline
  - Terrafom para generar la instancia obteniendo desde el terraform backend S3 la SG ID,VP ID y Key de la infraestructura Core.
  - Ansible para configurar el Servidor

Este Proyecto esta dividido en 3 repositorios, estos son:

  - https://github.com/ezequiellladoce/Grafana_Jenkinsfile.git donde se describe el depliege y donde se aloja el pipeline
  - https://github.com/ezequiellladoce/EC2_Grafana.git donde se esta el código terrafom para instalar la instancia
  - https://github.com/ezequiellladoce/Ansible_Grafana.git donde en la carpeta Output_ip se encuentra el Código para obtener la ip pública de la instancia  y en la carpeta Ansible  el playbook y los roles necesarios para la configuración de la Instancia.

## Pre-requisitos 📋

  - Infraestructura de red core creada de acuero al repositorio https://github.com/ezequiellladoce/Deploy_Jenkins_in_Created_Infra.git.

  - Servidor Jenkins creado de acuerdo la repositorio https://github.com/ezequiellladoce/Deploy_Jenkins_in_Created_Infra.git con las siguientes dependencias
    - TERRAFORM .12 o superior
    - AWS CLI
    - Ansible  
    - CUENTA FREE TIER AWS configurada en el AWS CLI

## Comenzando 🚀

### Descripción del pipeline (partes principales):    

#### Stage 'Get public ip'

Una ves que clonamos el repositorio https://github.com/ezequiellladoce/Ansible_Grafana.git entramos en la carpeta Output_ip y ejecutamos el terrafom con bash para obtener la ip publica de la instancia creada.

```
  dir('Output_ip'){
    sh '''
        #!/bin/bash
        ls
        pwd
        terraform --version
        terraform init
        terraform plan
        terraform apply -auto-approve
```
Luego creamos el achivo hosts y le asignamos la ip pública de la instancia recien creada, movemos el archivo hosts a la carpeta Ansible

```
        echo "[grafana_server]" > hosts
        terraform output pub_ip >> hosts
        terraform output pub_ip > server_ip
        mv hosts ../Ansible
```
#### Stage 'Configure instance'

Alli entramos en la carpeta Ansible y extraemos la clave del secretes manajer con el AWS Cli

```
aws secretsmanager get-secret-value --secret-id "ec2-key-c" --region "us-east-2" --query 'SecretString' --output text > key.pem

```
Luego ejecutamos el playbook

```
ansible-playbook playbook.yml -u ubuntu --key-file key.pem -i hosts

```

## Despliegue 📦

### Configuracion de Jenkins

Creamos en el sevidor Jenkins un nuevo Job y elejimos la opción pipeline, Luego  OK




Vamos hasta plipeline y configuramos:




- Definition        ----> Pipeline Script from GitSCM
- SCM               ----> Git
- Repository URL    ----> https://github.com/ezequiellladoce/Grafana_Jenkinsfile.git
- Branch Specifier  ----> */main
- Script Path       ----> grafana.jenkinsfile

Guardamos y corremos el Job

### Resultado Obtenido
