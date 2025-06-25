# 🐘 PostgreSQL + VS Code + Copilot Agent Setup Guide

> **Tutorial completo**: Configurar PostgreSQL con VS Code y Copilot Agent para desarrollo moderno sin fricción.

## 🎯 ¿Qué vas a lograr?

- ✅ Conectar PostgreSQL directo desde VS Code
- ✅ Usar Copilot Agent para generar SQL con lenguaje natural
- ✅ Trabajar con tus Bases de datos desde un solo lugar (no usar Apps externas como pgAdmin, DBiever, etc..)
- ✅ Workflow optimizado para DEVS con PostgreSQL (Odoo, Django, Laravel, etc.)

---

## 📋 Pre-requisitos

- [ ] **PostgreSQL instalado** y corriendo
- [ ] **VS Code instalado / Actualizado**

### Verificar PostgreSQL
```bash
# Verificar versión
psql --version
# Debería mostrar: psql (PostgreSQL) 15.x+

# Si no está instalado:
# macOS: brew install postgresql
# Ubuntu: sudo apt install postgresql postgresql-contrib  
# Windows: Descargar desde postgresql.org
```

---

## 🚀 Setup Paso a Paso

### 1️⃣ Crear Usuario Administrador Seguro

Linux
```bash
sudo -u postgres psql -c "CREATE USER odoo WITH PASSWORD 'Admin123' SUPERUSER CREATEDB;"
```
macOs
```bash
psql postgres -c "CREATE USER odoo WITH PASSWORD 'Admin123' SUPERUSER CREATEDB;"
```

Windows
```bash
psql -U postgres -d postgres -c "CREATE USER odoo WITH PASSWORD 'Admin123' SUPERUSER CREATEDB;"
```

**⚠️ Importante**: Cambia `Admin123` por tu password seguro.

### 2️⃣ Instalar Extensiones VS Code

#### PostgreSQL Extension (Microsoft)
1. Abrir VS Code
2. `Ctrl+Shift+X` (Extensions)
3. Buscar: `"PostgreSQL"`
4. Seleccionar: **PostgreSQL by Microsoft** (`ms-ossdata.vscode-pgsql`)
5. Click **Install**

#### GitHub Copilot (si no lo tienes)
1. En Extensions buscar: `"GitHub Copilot"`
2. Instalar: **GitHub Copilot**
3. Instalar: **GitHub Copilot Chat**
4. Login con tu cuenta GitHub

### 3️⃣ Configurar Copilot Agent

1. `Ctrl+,` (Abrir Settings)
2. Buscar: `"pgsql copilot"`
3. ✔️ Activar: **Pgsql › Copilot: Enable**
4. ✔️ Activar: **Pgsql › Copilot: Enable Agent Mode**
5. **Reload VS Code** (aparecerá popup automático)

### 4️⃣ Crear Primera Conexión

1. Click en 🐘 **PostgreSQL** (sidebar izquierda)
2. Click **"Add Connection"**
3. Completar formulario:
   ```
   Profile Name: Local Development
   Server: localhost
   Port: 5432
   Database: postgres
   Username: devadmin
   Password: [tu password del paso 1]
   ```
4. Click **Save & Connect**

---

## ✅ Verificación

Si todo está correcto deberías ver:
- [ ] 🐘 PostgreSQL icon en sidebar
- [ ] Conexión "Local Development" activa
- [ ] Puedes expandir schemas y tablas
- [ ] Copilot Chat responde a `@pgsql`

---

## 🎯 Primeras Pruebas

### Test 1: Query Básica
1. Abrir Copilot Chat (`Ctrl+Shift+I`)
2. Escribir: `@pgsql muestra todas las bases de datos`
3. Copilot debería generar y ejecutar: `SELECT datname FROM pg_database;`

### Test 2: Schema Exploration
```
@pgsql muéstrame todas las tablas en la base actual
```

### Test 3: Query con Datos
```
@pgsql muéstrame las últimas 5 filas de cualquier tabla con datos
```

---

## 🚀 Ejemplos Avanzados

### Para E-commerce
```
@pgsql muéstrame las ventas del último mes agrupadas por semana
```

### Para Odoo
```
@pgsql muéstrame pedidos de venta con cliente y comercial de los últimos 30 días
```

### Analytics
```
@pgsql crea un análisis de crecimiento mensual con porcentajes
```

---

## 🐞 Troubleshooting

### Error: "Extension initializing"
- Esperar 30-60 segundos tras instalación
- Reiniciar VS Code completamente

### Error: Copilot no responde
- Verificar que `@pgsql` esté habilitado en settings
- Verificar login en GitHub Copilot
- Reload VS Code

### Error: Connection refused
- Verificar que PostgreSQL esté corriendo:
  ```bash
  # macOS: brew services start postgresql
  # Ubuntu: sudo systemctl start postgresql
  # Windows: Ver Services > PostgreSQL
  ```

### Sin función @pgsql en chat
- Settings → `pgsql copilot` → Enable todas las opciones
- Reload VS Code obligatorio

---

## 💡 Tips Pro

### Comandos Útiles PostgreSQL
```bash
# Ver status PostgreSQL
brew services list | grep postgresql  # macOS
sudo systemctl status postgresql      # Ubuntu

# Conectar directamente
psql -U devadmin -h localhost -d postgres
```

### Shortcuts VS Code
- `Ctrl+Shift+X`: Extensions
- `Ctrl+,`: Settings  
- `Ctrl+Shift+I`: Copilot Chat
- `Ctrl+Shift+P`: Command Palette

### Prompts Efectivos para @pgsql
- Sé específico con fechas: "últimos 30 días", "este mes"
- Menciona agrupaciones: "por semana", "por cliente"
- Usa términos familiares: "ventas", "pedidos", "usuarios"

---

## 🎓 Próximos Pasos

1. **Prueba con tu proyecto real**: Conecta tu base de datos de desarrollo
2. **Explora schema visualization**: Right-click en database → "Visualize Schema"
3. **Combina con ORM**: Usa queries SQL para validar/optimizar tu código ORM
4. **Performance analysis**: Usa `@pgsql` para analizar queries lentas

---

## 🔗 Enlaces Útiles

- [Documentación oficial Microsoft](https://learn.microsoft.com/en-us/azure/postgresql/extensions/vs-code-extension/overview)
- [GitHub Repository](https://github.com/Microsoft/vscode-pgsql)
- [Campus Cleverit - Formación Odoo](https://campuscleverit.es)

---

## 📝 Feedback

¿Te funcionó este setup? ¿Encontraste algún problema?

- 👍 **Like** si te ayudó
- 💬 **Comenta** tu experiencia
- 🔄 **Comparte** con tu equipo

**Pregunta de reflexión**: ¿Crees que usar Copilot Agent nos hace mejores desarrolladores o más dependientes? 🤔

---

> **Creado por**: [Campus Cleverit](https://campuscleverit.es)  
> **Video tutorial**: [YouTube - JC Montoya](https://youtube.com/c/jcmontoya)
