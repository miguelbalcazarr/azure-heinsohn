# Creación de un Secret Scope en Azure Databricks con Azure Key Vault (RBAC)

## 📌 Descripción general
Un **Secret Scope** en Azure Databricks permite almacenar y gestionar secretos (credenciales, tokens, contraseñas, claves) de forma segura para que puedan ser utilizados en notebooks, jobs o clusters sin exponer datos sensibles en el código.

Cuando se integra con **Azure Key Vault**, la seguridad se gestiona desde **Azure RBAC (Role-Based Access Control)** y no desde políticas internas de Databricks, lo que asegura un control unificado y alineado con las mejores prácticas de seguridad de Azure.

---

## ✅ Requisitos previos
1. **Azure Key Vault**
   - Debe estar creado en el mismo **Resource Group** y **región** que el Workspace de Databricks.
   - Se deben habilitar permisos de acceso mediante **RBAC**.

2. **Roles de Azure**
   - La identidad administrada del **Workspace de Databricks** (o la UAMI asociada) debe tener como mínimo el rol:
     - **Key Vault Secrets User** sobre el Key Vault.
   - Opcionalmente, para administración avanzada:
     - **Key Vault Administrator** para configurar y gestionar el vault.

3. **Azure Databricks Workspace**
   - Debe estar en **Premium Plan** o superior para soportar integración con Key Vault.
   - Configurado con acceso mediante **Managed Identity**.

---

## ⚙️ Pasos de configuración

### 1. Crear o identificar un Azure Key Vault
- Desde el portal de Azure, crear un **Key Vault** o usar uno existente.
- Configurar acceso mediante **Azure RBAC** (no con políticas de acceso heredadas).
- Asignar a la identidad de Databricks el rol:
  - **Key Vault Secrets User** (mínimo requerido).

### 2. Crear el Secret Scope en Databricks
- Acceder al **Workspace de Databricks**.
- Ir al menú: **Data > Security > Secret Scopes**.
- Seleccionar **Create New Secret Scope**.
- Completar los parámetros:
  - **Scope Name:** nombre lógico para el scope (ejemplo: `kv-scope-dev`).
  - **Manage Principal:** seleccionar `users` o `admins` según la necesidad.
  - **Azure Key Vault DNS Name:** URL de tu Key Vault (ej. `https://kv-dbk-dev.vault.azure.net/`).
  - **Resource ID:** ID completo del Key Vault (ej. `/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<kv-name>`).

### 3. Validar el Secret Scope
Ejecutar en un notebook:
```python
dbutils.secrets.listScopes()
```

Verificar que el nuevo scope aparece en la lista.

### 4. Recuperar un secreto
Si en Key Vault existe el secreto `db-password`, puedes acceder a él:
```python
dbutils.secrets.get(scope="kv-scope-dev", key="db-password")
```

---

## 🎯 Beneficios de usar RBAC con Secret Scopes
- **Seguridad centralizada:** control de acceso gestionado desde Azure RBAC, sin depender de configuraciones locales.  
- **Cumplimiento normativo:** integración con estándares de seguridad de Azure y auditoría centralizada.  
- **Flexibilidad:** diferentes scopes pueden apuntar a distintos Key Vaults (ej. dev, qa, prod).  
- **Rotación de credenciales:** al rotar secretos en Key Vault, se reflejan automáticamente en Databricks.  

---

## 📚 Referencias
- [Azure Databricks – Secret Scopes](https://learn.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes)  
- [Integración de Key Vault con Databricks](https://learn.microsoft.com/en-us/azure/databricks/security/secrets/secrets-scopes#--azure-key-vault-backed-scopes)  
- [Asignación de roles con RBAC en Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/rbac-guide)  
