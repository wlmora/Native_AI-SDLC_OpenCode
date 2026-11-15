# Guía de Aprovisionamiento y Arranque: Ecosistema Native AI-SDLC

Esta guía estandariza la preparación desde cero de un entorno de desarrollo agéntico basado en **Ubuntu** (ya sea sobre **WSL 2 en Windows** o directamente en un **Servidor Virtual / VPS** en la nube) y define una estructura base agnóstica para cualquier proyecto de software.

---

## Parte 1: Preparación del Entorno en Windows (Solo si usas WSL 2)

Si vas a desplegar directamente en un servidor virtual con Ubuntu limpio, **salta a la Parte 2**.

### 1.1 Instalar o actualizar WSL con Ubuntu
Abre **PowerShell como Administrador** y ejecuta:

```powershell
# Instalar WSL con Ubuntu (en equipos nuevos)
wsl --install -d Ubuntu

# O actualizar el kernel de WSL a su última versión (si ya estaba instalado)
wsl --update
```

### 1.2 Optimizar memoria RAM y red en WSL
Para evitar que WSL consuma toda la memoria del equipo anfitrión y facilitar la comunicación de red local:

1. Abre el archivo de configuración global desde PowerShell:
   ```powershell
   notepad "\$env:USERPROFILE\.wslconfig"
   ```
2. Agrega la siguiente configuración (ajusta `memory` según la RAM total del equipo, ej. `8GB` en equipos de 16 GB):
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

### 2.1 Instalar dependencias esenciales y GitHub CLI de forma nativa en Linux
> **Nota importante para WSL:** Windows comparte su variable `PATH` con Ubuntu por defecto. Instalar `nodejs` y `npm` de forma nativa en Ubuntu (y refrescar con `hash -r`) evita que comandos como `npx` invoquen al `CMD.EXE` de Windows y fallen con errores de permisos sobre rutas UNC (`\\wsl.localhost\...`).

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl tar git nodejs npm gh
hash -r
```

### 2.2 Validar que los binarios apunten a Linux
Verifica que las herramientas estén operativas y que `npx` resuelva hacia `/usr/bin/npx` (nunca hacia `/mnt/c/...`):

```bash
git --version
node --version
npm --version
gh --version
which npx
```

### 2.3 Configurar identidad global estándar de Git
Configura la rama principal moderna (`main`) y las credenciales globales para que los agentes puedan realizar *commits* sin interrupciones:

```bash
git config --global init.defaultBranch main
git config --global user.name "wlmora"
git config --global user.email "wlmora@gmail.com"

# Verificar configuración aplicada
git config --list
```

### 2.4 Autenticar el entorno con GitHub (Una sola vez por equipo/servidor)
GitHub no permite contraseñas tradicionales en terminal. Autentica Git de forma segura mediante navegador o token usando GitHub CLI:

```bash
gh auth login
# Seleccionar: GitHub.com -> HTTPS -> Yes (autenticar Git) -> Login with a web browser
```

---

## Parte 3: Instalación y Validación de OpenCode

### 3.1 Instalar OpenCode
Ejecuta el instalador oficial (asegúrate de incluir la ruta completa `/install`):

```bash
curl -fsSL https://opencode.ai | bash
```

*(El script instala el binario en `~/.opencode/bin` y actualiza automáticamente `~/.bashrc` o `~/.zshrc`).*

### 3.2 Recargar sesión, actualizar y validar
Actualiza las variables de entorno de tu terminal y verifica la versión instalada:

```bash
source ~/.bashrc
opencode --version
```

*(Opcional: Al ejecutar `opencode` por primera vez, el cliente verifica y aplica automáticamente la última versión estable).*

---

## Parte 4: Estructura Base Agnóstica para Cualquier Proyecto

Para iniciar cualquier proyecto nuevo bajo metodología **Spec-Driven Development**, crea el directorio dentro del sistema de archivos nativo de Linux (`~/proyectos`) con esta estructura mínima:

```bash
# 1. Crear directorio del proyecto e inicializar repositorio Git
mkdir -p ~/proyectos/nombre-del-proyecto
cd ~/proyectos/nombre-del-proyecto
git init

# 2. Crear carpetas estándar de especificaciones, skills y reglas
mkdir -p docs/specs .opencode/skills
touch AGENTS.md opencode.json .gitignore

# 3. Configurar .gitignore base para evitar subir secretos o dependencias
cat << 'EOF' > .gitignore
node_modules/
dist/
.astro/
.env
.env.*
!.env.example
.DS_Store
EOF
```

### Propósito de cada archivo en la estructura base
*   **`AGENTS.md` (Constitución del Proyecto):** Define el objetivo del producto, el *stack* tecnológico, las reglas de negocio y el flujo obligatorio de planificar antes de escribir código. OpenCode lo lee en cada sesión.
*   **`opencode.json` (Servidores MCP):** Conecta a OpenCode con fuentes de documentación viva (ej. MCP oficial de Astro, Supabase o Context7) para evitar código obsoleto.
*   **`.agents/skills/` y `.opencode/skills/` (Habilidades bajo demanda):** Manuales especializados (`SKILL.md`) que el agente carga solo cuando trabaja en una tarea concreta. Puedes instalar skills oficiales por proyecto ejecutando:
    ```bash
    /usr/bin/npx skills add <propietario/repositorio> --skill <nombre-skill>
    # En el asistente interactivo: Scope -> Project | Method -> Copy
    ```
*   **`docs/specs/` (Memoria Evolutiva):** Contiene las especificaciones técnicas incrementales (`01-mvp-arquitectura.md`, `02-modulo-pagos.md`, etc.) redactadas y actualizadas por OpenCode antes de codificar.

---

## Parte 5: Versionamiento y Subida a GitHub

Una vez configurados los cimientos o completada una especificación, respalda el proyecto en GitHub siguiendo estos pasos:

### 5.1 Crear el repositorio remoto (Desde terminal con `gh` o en la web)
```bash
# Opción rápida: Crear el repositorio en GitHub y vincularlo automáticamente
gh repo create nombre-del-proyecto --private --source=. --remote=origin
```
*(Si lo creaste manualmente en la web como repositorio vacío, vincúlalo con `git remote add origin https://github.com`. Si necesitas corregir una URL mal escrita, usa `git remote set-url origin <nueva-url>`).*

### 5.2 Registrar el commit y subir cambios
```bash
# 1. Verificar archivos listos para versionar (asegúrate de que .env no aparezca)
git status

# 2. Preparar archivos y crear el commit
git add .
git commit -m "chore: inicializa estructura base Native AI-SDLC, reglas y skills"

# 3. Subir la rama main al repositorio remoto
git push -u origin main
```

---

## Parte 6: Ciclo de Trabajo Diario en OpenCode

1. **Inicia el agente** dentro de tu proyecto: `opencode`
2. **Modo Planificación (`Tab`):** Describe el nuevo requisito de negocio y pídele que redacte o actualice la especificación en `docs/specs/`.
3. **Revisión Humana:** Lee el documento Markdown generado y solicita ajustes si es necesario.
4. **Modo Construcción (`Tab`):** Aprueba el plan para que el agente implemente el código, ejecute las validaciones (`npm run check`, `npm run build`) y deje los cambios listos para tu siguiente `git commit` y `git push`.
