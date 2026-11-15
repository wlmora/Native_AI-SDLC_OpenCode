# Guía de Aprovisionamiento: Entorno Native AI-SDLC con OpenCode

Esta guía estandariza la preparación desde cero de un entorno de desarrollo agéntico basado en **Ubuntu** (ya sea sobre **WSL 2 en Windows** o directamente en un **Servidor Virtual / VPS** en la nube).

---

## Parte 1: Preparación del Sistema Operativo (Solo si usas Windows)

Si vas a desplegar directamente en un servidor virtual con Ubuntu, **salta a la Parte 2**. Si estás en una estación de trabajo con Windows 10/11, configura WSL 2 antes de continuar.

### 1.1 Instalar o actualizar WSL con la última versión de Ubuntu
Abre **PowerShell como Administrador** y ejecuta:

```powershell
# Instalar WSL con Ubuntu (si es un equipo nuevo)
wsl --install -d Ubuntu

# O actualizar el kernel de WSL a su última versión (si ya estaba instalado)
wsl --update
```

### 1.2 Optimizar recursos de memoria y red en WSL
Para evitar que WSL consuma toda la memoria RAM del equipo anfitrión y facilitar la comunicación de red local:

1. Abre el archivo de configuración global desde PowerShell:
   ```powershell
   notepad "\$env:USERPROFILE\.wslconfig"
   ```
2. Agrega la siguiente configuración (ajusta `memory` según la capacidad del equipo, ej. `8GB` para equipos de 16 GB):
   ```ini
   [wsl2]
   memory=8GB
   networkingMode=mirrored
   ```
3. Guarda el archivo y reinicia WSL para aplicar los cambios:
   ```powershell
   wsl --shutdown
   ```

---

## Parte 2: Aprovisionamiento Base en Ubuntu (WSL o Servidor Virtual)

Ejecuta estos pasos dentro de tu terminal de Ubuntu.

### 2.1 Actualizar el sistema e instalar dependencias esenciales
OpenCode y los flujos agénticos requieren `curl`, `tar` y `git` para descargar binarios, descomprimir paquetes y gestionar versiones:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl tar git
```

### 2.2 Validar e inicializar la configuración estándar de Git
Verifica que Git esté operativo y configura la identidad global y la rama principal por defecto (`main`) para que los agentes puedan realizar *commits* sin errores:

```bash
# 1. Validar instalación
git --version

# 2. Configurar rama principal moderna y credenciales globales
git config --global init.defaultBranch main
git config --global user.name "wlmora"
git config --global user.email "wlmora@gmail.com"

# 3. Verificar configuración aplicada
git config --list
```

---

## Parte 3: Instalación y Validación de OpenCode

### 3.1 Instalar OpenCode
Descarga y ejecuta el instalador oficial (asegúrate de incluir la ruta completa `/install`):

```bash
curl -fsSL https://opencode.ai | bash
```

*(Nota: El script instala el binario en `~/.opencode/bin` y añade automáticamente la ruta a tu archivo `~/.bashrc` o `~/.zshrc`).*

### 3.2 Recargar entorno y validar versión
Actualiza las variables de sesión de tu terminal y verifica que el comando responda correctamente:

```bash
source ~/.bashrc
opencode --version
```

### 3.3 Actualizar a la última versión disponible
Al ejecutar `opencode` por primera vez (o mediante su comando de actualización), el cliente verifica y aplica la última versión estable (ej. `v2.3.0` o superior):

```bash
opencode
# Sal con Ctrl+C y verifica la versión actualizada:
opencode --version
```

*(Opcional para servidores de producción: Si necesitas fijar una versión exacta al aprovisionar, usa `curl -fsSL https://opencode.ai | bash -s -- --version X.Y.Z`).*

---

## Parte 4: Inicialización de un Nuevo Proyecto AI-SDLC

Para cada proyecto nuevo que vayas a desarrollar con agentes, sigue esta estructura estándar dentro del sistema de archivos nativo de Linux (`~/proyectos`):

```bash
# 1. Crear directorio del proyecto e inicializar repositorio
mkdir -p ~/proyectos/nombre-del-proyecto
cd ~/proyectos/nombre-del-proyecto
git init

# 2. Crear archivo base de reglas y arquitectura para los agentes
touch AGENTS.md

# 3. Iniciar el entorno agéntico
opencode
```

### Flujo básico de operación dentro de OpenCode
* **Conectar proveedores externos (Opcional):** OpenCode incluye modelos gratuitos listos para usar. Si deseas añadir una API Key propia (Google AI Studio, OpenRouter, etc.), ejecuta `/connect` dentro de la interfaz.
* **Modo Planificación (`Tab`):** Úsalo siempre al iniciar una funcionalidad para que el agente analice el contexto, redacte especificaciones en `AGENTS.md` y proponga un plan sin modificar código.
* **Modo Construcción (`Tab`):** Úsalo una vez aprobado el plan para que el agente implemente el código y ejecute las pruebas correspondientes.
