# Requisitos Mínimos para Despliegue en VPS Compartido

## Resumen Ejecutivo
Este documento detalla los requerimientos mínimos necesarios para ejecutar el proyecto **mini-POS-AdminLTE** en un entorno VPS (Virtual Private Server) compartido.

---

## 📋 Requisitos del Servidor

### 1. Sistema Operativo
- **Windows Server 2012 R2 o superior** (recomendado: Windows Server 2016/2019/2022)
- O cualquier sistema operativo compatible con IIS 8.0 o superior

### 2. Servidor Web
- **IIS (Internet Information Services) 8.0 o superior**
- Módulos requeridos:
  - ASP.NET 4.6.2 o superior
  - WebSockets (para funcionalidades en tiempo real)
  - URL Rewrite Module (opcional, pero recomendado)

### 3. Framework .NET
- **.NET Framework 4.6.2 o superior** (instalado en el servidor)
- **ASP.NET MVC 5.2.7**
- **ASP.NET Web API 5.2.7**

---

## 🗄️ Requisitos de Base de Datos

### Microsoft SQL Server
- **SQL Server 2012 o superior** (recomendado: SQL Server 2016+)
- O **SQL Server Express** (versión gratuita, suficiente para proyectos pequeños)
- **Espacio mínimo**: 500 MB para la base de datos
- **Permisos requeridos**:
  - Crear bases de datos
  - Crear tablas
  - Ejecutar procedimientos almacenados
  - SELECT, INSERT, UPDATE, DELETE

**Nota**: El proyecto utiliza Entity Framework 6.4.4 con Code First Migrations, por lo que la base de datos se puede crear automáticamente.

---

## 🔧 Requisitos de Desarrollo y Compilación

### Node.js (Para compilación de assets frontend)
- **Node.js 10.x o superior** (recomendado: 14.x o 16.x)
- **NPM 6.x o superior**

### Paquetes NuGet Principales
Los siguientes paquetes son descargados automáticamente durante la restauración:
- Entity Framework 6.4.4
- Autofac 5.0.0 (Inyección de dependencias)
- Microsoft.Owin 4.1.0 (Middleware OWIN)
- Newtonsoft.Json 12.0.2
- Microsoft.AspNet.WebApi 5.2.7

---

## 💾 Requisitos de Recursos del Servidor

### Mínimos Recomendados para VPS Compartido:
- **CPU**: 1 vCore (mínimo)
- **RAM**: 2 GB (mínimo) - 4 GB (recomendado)
- **Almacenamiento**: 5 GB de espacio libre
- **Ancho de banda**: 1 TB/mes (para tráfico moderado)

### Puertos Requeridos:
- **Puerto 80** (HTTP)
- **Puerto 443** (HTTPS) - Recomendado con certificado SSL

---

## 📦 Dependencias Frontend (AdminLTE)

Las siguientes librerías se instalan automáticamente con `npm install`:
- Bootstrap 4.4.1
- jQuery 3.4.1
- Font Awesome 5.13.0
- Chart.js 2.9.3
- DataTables 1.10.20
- Select2 4.0.13
- Moment.js 2.24.0
- SweetAlert2 9.10.8
- Y otras dependencias del paquete AdminLTE

---

## ⚙️ Configuración Requerida

### 1. Cadena de Conexión (connectionStrings.config)
Ubicación: `Configs/connectionStrings.config`

```xml
<connectionStrings>
  <add name="DefaultContext" 
       providerName="System.Data.SqlClient" 
       connectionString="Data Source=TU_SERVIDOR;Initial Catalog=miniPOS;User ID=TU_USUARIO;Password=TU_PASSWORD" />
</connectionStrings>
```

### 2. Configuración de Aplicación (appSettings.config)
Ubicación: `Configs/appSettings.config`

Los valores predeterminados son adecuados para la mayoría de los casos. Modifica solo si es necesario.

### 3. Permisos de Carpetas
El usuario de IIS (generalmente `IIS_IUSRS` o `NETWORK SERVICE`) necesita permisos de:
- **Lectura**: En toda la aplicación
- **Lectura/Escritura**: En la carpeta `App_Data` (si se usa)

---

## 🚀 Proceso de Despliegue en VPS Compartido

### Paso 1: Preparar el Entorno
```bash
# En tu máquina local o servidor de compilación
git clone https://github.com/damartinezisa/mini-POS-AdminLTE.git
cd mini-POS-AdminLTE
```

### Paso 2: Compilar Assets Frontend
```bash
cd WebCore
npm install
npm run compile
cd ..
```

### Paso 3: Compilar la Aplicación
1. Abrir `AspMVCAdminLTE.sln` en Visual Studio
2. Restaurar paquetes NuGet (clic derecho en solución → "Restore NuGet Packages")
3. Compilar en modo **Release** (Build → Configuration Manager → Release)
4. Publicar la aplicación (clic derecho en proyecto → Publish)

### Paso 4: Configurar en el VPS
1. **Subir archivos** al VPS mediante FTP/SFTP o Web Deploy
2. **Crear base de datos** SQL Server con el nombre deseado
3. **Actualizar** `connectionStrings.config` con la cadena de conexión correcta
4. **Configurar sitio en IIS**:
   - Crear nuevo sitio o aplicación
   - Apuntar al directorio donde están los archivos
   - Configurar Application Pool con .NET CLR v4.0
   - Modo de pipeline: Integrated

### Paso 5: Ejecutar Migraciones
Si usas Entity Framework Migrations:
```bash
# Desde Package Manager Console en Visual Studio conectado al VPS
Update-Database
```

O configurar en `Web.config` para migración automática (no recomendado en producción):
```csharp
Database.SetInitializer(new MigrateDatabaseToLatestVersion<RepositoryContext, Configuration>());
```

---

## 🔐 Consideraciones de Seguridad

### Recomendaciones Mínimas:
1. **Cambiar la clave de encriptación** en `appSettings.config`:
   ```xml
   <add key="EncryptionKey" value="TU_CLAVE_SEGURA_DE_16_CHARS" />
   ```

2. **Usar HTTPS**: Instalar certificado SSL (Let's Encrypt es gratuito)

3. **Cambiar credenciales predeterminadas** de la base de datos

4. **Deshabilitar errores detallados** en producción:
   ```xml
   <system.web>
     <customErrors mode="On" />
   </system.web>
   ```

5. **Mantener actualizado** el framework y dependencias

---

## 📊 Funcionalidades Incluidas

El proyecto incluye:
- ✅ Sistema de autenticación con tokens (OWIN)
- ✅ Web API con autenticación Bearer Token
- ✅ Panel administrativo con AdminLTE
- ✅ Entity Framework 6 con patrón Repository
- ✅ Inyección de dependencias con Autofac
- ✅ Soporte para CORS

---

## 🔍 Verificación Post-Despliegue

Después del despliegue, verifica:
1. **Acceso al sitio**: `http://tu-dominio.com` o `https://tu-dominio.com`
2. **Acceso a la base de datos**: Verifica la conexión y que las tablas se hayan creado
3. **Assets frontend**: Verifica que CSS y JavaScript cargan correctamente
4. **API**: Prueba endpoints en `/api/` (ejemplo: Postman)
5. **Logs**: Revisa Event Viewer de Windows o logs de IIS para errores

---

## 🆘 Solución de Problemas Comunes

### Error: "Could not load file or assembly"
- **Solución**: Verificar que .NET Framework 4.6.2 esté instalado en el servidor

### Error 500.19: "Cannot read configuration file"
- **Solución**: Verificar permisos de lectura en `Web.config`

### Error de conexión a base de datos
- **Solución**: 
  - Verificar cadena de conexión en `connectionStrings.config`
  - Verificar firewall de SQL Server
  - Verificar credenciales y permisos de usuario

### Assets (CSS/JS) no cargan
- **Solución**: 
  - Verificar que se ejecutó `npm run compile` en WebCore
  - Verificar que la carpeta `dist` existe con los archivos compilados

---

## 📞 Soporte y Documentación Adicional

- **Repositorio**: https://github.com/damartinezisa/mini-POS-AdminLTE
- **AdminLTE**: https://adminlte.io/docs
- **ASP.NET MVC**: https://docs.microsoft.com/aspnet/mvc
- **Entity Framework 6**: https://docs.microsoft.com/ef/ef6

---

## 📝 Notas Finales

- Este proyecto está basado en ASP.NET MVC tradicional (.NET Framework), **no** en .NET Core/.NET 5+
- Para VPS con Linux, se requeriría migrar a .NET Core, ya que .NET Framework solo funciona en Windows
- El uso de SQL Server puede ser reemplazado por otras bases de datos compatibles con Entity Framework 6, pero requiere modificaciones en el código

---

**Fecha de actualización**: Noviembre 2025  
**Versión del documento**: 1.0
