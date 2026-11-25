# Guía de Skills, MCPs e Inteligencia de Contexto para OpenCode

En un flujo **Native AI-SDLC**, el mayor enemigo de la calidad del código (especialmente al usar modelos gratuitos) es la **saturación de contexto** y el **código obsoleto**. Si escribes miles de líneas de reglas en `AGENTS.md`, el modelo pierde precisión y gasta tokens innecesarios en cada mensaje.

Esta guía estandariza la **arquitectura de conocimiento bajo demanda** (*Progressive Disclosure*) para que OpenCode consulte únicamente las reglas, documentación y dependencias exactas que necesita en cada tarea.

---

## 1. Arquitectura de Conocimiento: ¿Qué hace cada pieza?

| Herramienta / Archivo | Ubicación en el Proyecto | Cuándo lo usa OpenCode | Propósito |
| :--- | :--- | :--- | :--- |
| **`AGENTS.md`** | Raíz (`/AGENTS.md`) | **Siempre** (en cada prompt) | Constitución corta del proyecto: stack, roles de negocio, reglas de seguridad y flujo de trabajo. |
| **Agent Skills** | `.agents/skills/` y `.opencode/skills/` | **Bajo demanda** (solo cuando la tarea coincide) | Manuales técnicos especializados (`SKILL.md`) que evitan malas prácticas sin saturar la memoria base. |
| **Servidores MCP** | `opencode.json` | **En tiempo real** (al consultar docs o grafos) | Conexión directa a documentación oficial actualizada o herramientas externas. |
| **CodeGraph** | `.codegraph/` (local, ignorado en Git) | **Al explorar o modificar código** | Grafo de dependencias indexado en Rust que entrega el código exacto sin leer archivo por archivo. |

---

## 2. Catálogo de Skills Recomendadas (Web & Arquitectura)

### 2.1 Skills Oficiales del Ecosistema (Instaladas vía CLI)

> **Recordatorio para WSL:** Ejecuta siempre `/usr/bin/npx` (con ruta absoluta de Linux) para evitar que WSL invoque el `npx` de Windows y falle por permisos de rutas UNC.

1. **`astro-developer` (Oficial de Astro)**
   * **Qué aporta:** Patrones modernos de enrutamiento, separación entre páginas estáticas (SSG) y renderizado bajo demanda (SSR), manejo de activos y reglas estrictas para mantener **Zero-JS por defecto** hidratando islas (Vue/React) solo cuando es necesario.
   * **Comando de instalación:**
     ```bash
     /usr/bin/npx skills add withastro/astro --skill astro-developer
     ```

2. **`modern-web-guidance` (Oficial de Google Chrome)**
   * **Qué aporta:** Más de 200 guías evaluadas por los equipos de Chrome, Edge, Firefox y MDN. Obliga al agente a usar APIs nativas modernas del navegador (`<dialog>`, Popover API, CSS Container Queries, colores `oklch`, manejo moderno de fechas con `Temporal` y optimización de Core Web Vitals LCP/INP) en lugar de instalar librerías pesadas o escribir JavaScript innecesario.
   * **Comando de instalación:**
     ```bash
     /usr/bin/npx skills add GoogleChrome/modern-web-guidance
     ```
   * *(Opcional: Para desactivar la telemetría anónima de uso de guías, agrega `export DISABLE_TELEMETRY=1` en tu `~/.bashrc`).*

3. **`find-skills` (Meta-Skill de Descubrimiento)**
   * **Qué aporta:** Permite que el propio agente busque y sugiera nuevas skills oficiales en el registro abierto cuando necesites integrar una tecnología nueva (ej. pasarelas de pago, pruebas E2E o Supabase).
   * **Comando de instalación:**
     ```bash
     /usr/bin/npx skills add vercel-labs/skills --skill find-skills
     ```

*(Nota: En todos los asistentes interactivos de `npx skills add`, selecciona **Scope -> Project** para que la skill quede versionada dentro del repositorio y **Method -> Copy** para portabilidad entre WSL y servidores Linux).*

---

### 2.2 Skills Personalizadas del Proyecto (`.opencode/skills/`)

Para reglas de negocio o diseño propias que no existen en repositorios públicos, se crean carpetas locales dentro de `.opencode/skills/<nombre-skill>/SKILL.md` con cabecera YAML (`name` y `description`).

* **Ejemplo: `landing-page-design`**
  * **Propósito:** Evitar estética genérica generada por IA ("AI Slop": fuentes genéricas sin personalidad, gradientes morados repetitivos o exceso de iconos decorativos), imponiendo estructura de conversión modular, jerarquía visual clara y tokens semánticos centralizados.

---

## 3. Servidores MCP e Indexación de Código

### 3.1 Documentación Viva de Astro (`opencode.json`)
Permite que OpenCode consulte la documentación oficial de Astro en tiempo real antes de escribir código, evitando configuraciones obsoletas de versiones anteriores:

```json
{
  "\$schema": "https://opencode.ai",
  "mcp": {
    "astro-docs": {
      "type": "remote",
      "url": "https://astro.build",
      "enabled": true
    }
  }
}
```

### 3.2 CodeGraph (Inteligencia Estructural 100% Local)
A medida que el proyecto crece, explorar archivos uno por uno consume muchos tokens. **CodeGraph** construye una base de datos SQLite local impulsada por Rust que mapea símbolos, rutas de Astro, componentes Vue y relaciones de impacto.

```bash
# 1. Instalar y conectar el servidor MCP de CodeGraph en OpenCode (nivel proyecto)
/usr/bin/npx @colbymchenry/codegraph install --target=opencode --location=local --yes

# 2. Construir el índice inicial del repositorio
codegraph init --yes

# 3. Ignorar la base de datos local en Git (se reconstruye en segundos en cualquier máquina)
echo ".codegraph/" >> .gitignore
```

* **Comandos útiles de mantenimiento:**
  * `codegraph status`: Verifica si el índice está sincronizado.
  * `codegraph index`: Reconstruye el grafo completo tras cambios masivos de ramas.

---

## 4. Criterios de Gobernanza (Qué evitar al iniciar)

Para mantener el ecosistema rápido y predecible (especialmente con modelos gratuitos):
1. **No duplicar archivos de reglas:** Usar únicamente `AGENTS.md` en la raíz; no crear `.opencoderules` paralelos con instrucciones contradictorias.
2. **Evitar librerías UI en fase Beta:** Priorizar estándares web nativos (`modern-web-guidance`) y componentes propios con Tailwind CSS v4 antes de introducir SDKs experimentales.
3. **Mantener las skills ligeras:** Instalar únicamente las skills del *stack* activo del proyecto.
