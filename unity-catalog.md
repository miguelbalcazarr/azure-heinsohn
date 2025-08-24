# Configuración de Unity Catalog en Azure Databricks

## 📌 Descripción general
Unity Catalog es el sistema de gobernanza de datos de Databricks que permite la **gestión centralizada de datos, permisos y auditoría** dentro de un Data Lakehouse.  
En Azure Databricks, Unity Catalog se integra con **Azure Active Directory (AAD)** y **Azure Data Lake Storage Gen2 (ADLS Gen2)** para habilitar un modelo seguro de acceso basado en roles.

---

## 🏗️ Arquitectura de referencia
- **Control Plane:** Administrado por Databricks (metadatos, configuración).
- **Data Plane:** Almacenamiento en ADLS Gen2 dentro de tu suscripción de Azure.
- **Identidad:** Autenticación y autorización a través de Azure Active Directory.
- **Metastore:** Definición central de catálogos, esquemas y tablas.

---

## ✅ Requisitos previos

Antes de iniciar, asegúrate de que los **recursos principales** estén creados dentro de un mismo **Resource Group en Azure**, para mantener la trazabilidad y simplificar la administración.

1. **Rol del usuario configurador**
   - El usuario que realizará la configuración debe tener el rol de **Administrador de la suscripción de Azure**.
   - Este rol es necesario para:
     - Crear y asignar identidades administradas.
     - Configurar permisos en ADLS Gen2.
     - Desplegar y vincular el Conector de acceso.

2. **Azure Active Directory**
   - Inquilino habilitado con usuarios y grupos.
   - Aplicación de servicio registrada para Databricks.

3. **Azure Storage (ADLS Gen2)**
   - Contenedor creado para servir como almacenamiento principal.
   - Configuración de **RBAC** para la identidad administrada asociada al conector.  
   - Opcionalmente, aplicar **ACLs** adicionales a nivel de carpeta o archivo según políticas de seguridad.

4. **Azure Databricks Workspace**
   - Debe estar en **Premium Plan** o superior (Unity Catalog no disponible en Standard).
   - Configuración con acceso a **Managed Identity**.

5. **Conector de acceso para Azure Databricks**
   - Necesario para la integración de Unity Catalog con ADLS Gen2.
   - Se debe desplegar desde el portal de Azure dentro del **mismo Resource Group**.
   - Durante su configuración se debe asociar una **Managed Identity** (preferentemente UAMI).

6. **Identidad administrada para la autenticación**
   - Unity Catalog requiere una **Managed Identity** para acceder a ADLS Gen2.
   - Se soportan dos tipos:
     - **System-assigned managed identity (SAMI):** se crea automáticamente y se asocia a un recurso específico (menos flexible).
     - **User-assigned managed identity (UAMI):** se crea como recurso independiente en Azure y puede asociarse a múltiples recursos (**recomendada**).
   - La identidad administrada asociada al conector debe tener el rol:
     - **`Storage Blob Data Contributor`** en la **cuenta de almacenamiento (ADLS Gen2)**.

7. **Permisos necesarios adicionales**
   - Permisos de administrador de metastore para configurar Unity Catalog dentro de Databricks.

> ⚠️ **Nota importante**: El **ADLS Gen2**, el **Workspace de Databricks**, el **Conector de acceso** y la **Managed Identity (UAMI o SAMI)** deben estar en la **misma región** y dentro del **mismo Resource Group**.

---

## ⚙️ Pasos de configuración

### 1. Crear un Metastore en Unity Catalog
- Ingresar a [Azure Databricks Account Console](https://accounts.azuredatabricks.net/)
- En caso no se pueda ingresar, ir al **paso 4**.
- Navegar a **Unity Catalog > Metastores**.
- Crear un nuevo Metastore asignando:
  - **Nombre**
  - **Región**
  - **Ruta ADLS Gen2** `abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/<path>`
  - **ID del conector de acceso** `/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.Databricks/accessConnectors/<connector-name>`

### 2. Configurar el almacenamiento
- En el contenedor de ADLS, asignar permisos a:
  - **Managed Identity asociada al Conector de acceso**.
  - **Usuarios o grupos de AAD** según las políticas de acceso.
- Validar que las ACLs de ADLS permiten lectura/escritura (si son necesarias según la política).

### 3. Asignar el Metastore al Workspace
- En la consola de administración, asociar el **Metastore creado** con el **Workspace de Databricks**.
- Definir la región de operación.

### 4. Si el usuario configurador no tiene permisos de administrador
- En caso de que el usuario que vaya a configurar **Unity Catalog** no tenga el rol de **Administrador en la suscripción**, se debe asignar el rol de **Account Admin** en la cuenta de Databricks.
- Para ello:
  - Un usuario con permisos de administrador debe acceder a: [Azure Databricks Account Console](https://accounts.azuredatabricks.net/)
  - Asignar el rol de **Account Admin** al usuario que realizará la configuración.
- De esta forma, el usuario podrá continuar con la configuración de Unity Catalog sin necesidad de permisos directos sobre la suscripción.

---

## 📚 Referencias
- [Guía de inicio con Unity Catalog en Azure Databricks](https://learn.microsoft.com/es-es/azure/databricks/data-governance/unity-catalog/get-started)  
- [Configuración de almacenamiento ADLS para Unity Catalog](https://learn.microsoft.com/es-es/azure/databricks/data-governance/unity-catalog/storage)  
- [Managed Identities con Unity Catalog](https://learn.microsoft.com/es-es/azure/databricks/connect/unity-catalog/cloud-storage/azure-managed-identities)  
- [Permisos en Unity Catalog](https://learn.microsoft.com/es-es/azure/databricks/data-governance/unity-catalog/manage-privileges/)  
