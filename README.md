# 🧩 Reto 2 — Configuración con Ansible

> Automatización de la configuración de infraestructura en AWS usando **Ansible**.  
> Infraestructura creada manualmente; la configuración y despliegue se realizan con *playbooks*.

---

## ☁️ Arquitectura (resumen)

La solución usa:

- Instancias **EC2** etiquetadas para que el inventario dinámico las seleccione.  
- Una **instancia manager** que ejecuta los playbooks (ej. `main.yml`).  
- **RDS** (MySQL/MariaDB) para la base de datos.  
- **Ansible Vault** + **AWS Secrets Manager** para manejo seguro de secretos.

Imagen (referencia):

![Infraestructura AWS](https://github.com/user-attachments/assets/6462d480-072b-4c4f-9d82-e50d6a5630ba)

---

## ⚙️ Resumen de responsabilidades

- Ansible: instalación de dependencias, configuración de instancias y despliegue de la app Flask.  
- Playbooks: creación de DB, creación de usuarios DB, despliegue y configuración del servicio.  
- Vault + Secrets Manager: proteger credenciales sensibles (no hardcodear contraseñas).

---

## 🧭 Flujo de trabajo — pasos principales

### 1) Preparar variables sensibles (`vault.yml`)

Crear el archivo en:

```
reto-2-ansible/group_vars/all/vault.yml
```

Ejemplo de contenido (antes de cifrar):

```yaml
db_name: nombre_de_tu_base
db_login_host: endpoint_de_rds
db_user_admin: admin
db_admin_password: contraseña_admin
db_new_user: usuario_app
db_new_user_password: contraseña_usuario
db_port: 3306
```

> **No** subir `vault.yml` sin cifrar al repositorio.

---

### 2) Recuperar la clave del Vault desde AWS Secrets Manager

Se suministra un playbook auxiliar `get_vault_password.yml` que:

- Lee el secreto llamado `draconreyes_secret` en **AWS Secrets Manager**.  
- Guarda el resultado en `reto-2-ansible/vaultkey.txt`.

El secreto en AWS debe tener la estructura JSON esperada (ejemplo visual):

![Estructura JSON del secreto](https://github.com/user-attachments/assets/4d44fcb7-40d3-40f7-b8ea-3c8247abd07c)

> `get_vault_password.yml` facilita la integración entre Secrets Manager y Ansible Vault.

---

### 3) Configurar `ansible.cfg`

Ejemplo recomendado de `ansible.cfg` (ubicado en la raíz del repositorio):

```ini
[defaults]
inventory = inventario.aws_ec2.yml
vault_password_file = vaultkey.txt
vault_identity_list = default@vaultkey.txt

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

- `vault_password_file` apunta al archivo que crea `get_vault_password.yml`.  
- `vault_identity_list` permite usar identificadores de vault si trabajas con múltiples identidades.

---

### 4) Cifrar `vault.yml`

Con la `vaultkey.txt` en su lugar puedes cifrar:

```bash
ansible-vault encrypt --encrypt-vault-id default group_vars/all/vault.yml
```

Ejemplo del resultado (interfaz):

![Cifrado Vault](https://github.com/user-attachments/assets/11efd3e1-b622-4cc2-aa71-34a658c0a5e9)

---

### 5) Etiquetar las instancias en AWS

Para que el inventario dinámico (EC2) seleccione las máquinas correctas, añade estas etiquetas a las instancias:

| Key         | Value     |
|-------------|-----------|
| Application | app       |
| Name        | host-reto |

![Etiquetas AWS](https://github.com/user-attachments/assets/ce578864-08ed-4294-a254-4906a3bcf88b)

> El `inventario.aws_ec2.yml` utiliza etiquetas para filtrar instancias. Asegúrate de que las etiquetas coincidan exactamente.

---

### 6) Ejecutar el playbook principal

Una vez todo listo (variables cifradas, vaultkey disponible, instancias etiquetadas), ejecuta:

```bash
ansible-playbook main.yml --verbose
```

`main.yml` realiza, entre otras tareas:

- Conexión SSH a las EC2.
- Instalación de dependencias del servidor (Python, pip, Nginx, etc.).
- Despliegue y configuración de la app Flask.
- Creación de la base de datos y usuarios en RDS.
- Configuración de servicios y recursos auxiliares.



**Autor:** Diego Reyes  
**Proyecto:** Reto 2 — Automatización con Ansible

