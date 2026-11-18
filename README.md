# ansible-nginx-role

Rol de Ansible listo para portafolio que instala y configura Nginx en dos hosts locales basados en Docker. Incluye una plantilla Jinja2, secretos cifrados con `ansible-vault` y pruebas automatizadas con Molecule.

## Qué incluye

- Rol `nginx` con tareas, handlers, metadatos y valores por defecto.
- Plantilla `nginx.conf.j2` parametrizada con Jinja2.
- Variable sensible cifrada con `ansible-vault` en `roles/nginx/vars/vault.yml`.
- Inventario con dos hosts: `nginx01` y `nginx02`.
- Escenario Molecule con dos hosts efímeros independientes para converge y verify.
- Imagen Docker base construida desde `Dockerfile`.

## Estructura

```text
.
├── ansible.cfg
├── Dockerfile
├── docker-compose.yml
├── inventory.ini
├── requirements-dev.txt
├── requirements.yml
├── roles/nginx
│   ├── defaults/main.yml
│   ├── handlers/main.yml
│   ├── meta/main.yml
│   ├── tasks/main.yml
│   ├── templates/nginx.conf.j2
│   └── vars/vault.yml
├── molecule/default
│   ├── converge.yml
│   ├── molecule.yml
│   ├── requirements.yml
│   └── verify.yml
└── site.yml
```

## Requisitos

- macOS o Linux con Docker en ejecución.
- Python 3.11 o superior.
- Ansible 11 o superior.

## Preparación

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
```

La contraseña de Vault vive en `.vault_pass` y está excluida de Git. El archivo cifrado incluido usa una contraseña de demostración que no se comparte; al clonar el repo, genera la tuya y vuelve a crear el secreto cifrado:

```bash
python3 -c "import secrets; print(secrets.token_hex(24))" > .vault_pass
chmod 600 .vault_pass
printf '%s\n' 'nginx_response_message: "Hola desde Ansible Vault"' > roles/nginx/vars/vault.yml
ansible-vault encrypt roles/nginx/vars/vault.yml
```

Para inspeccionar el contenido cifrado:

```bash
ansible-vault view roles/nginx/vars/vault.yml
```

## Despliegue en los dos hosts

Construye la imagen base y levanta los contenedores:

```bash
docker build -t ansible-nginx-test:latest .
docker compose up -d
```

Aplica el rol:

```bash
ansible-playbook site.yml
```

Comprueba el resultado:

```bash
curl http://localhost:8081
curl http://localhost:8082
```

Ambos deben responder con el mensaje definido en el archivo cifrado de Vault.

## Pruebas con Molecule

Molecule crea y destruye sus propios contenedores, así que no depende de los contenedores anteriores:

```bash
molecule test
```

El escenario ejecuta `converge.yml` y luego `verify.yml`, que valida la sintaxis de Nginx y que la plantilla Jinja2 renderizó el valor cifrado.

## Limpieza

```bash
docker compose down
docker rmi ansible-nginx-test:latest
```
