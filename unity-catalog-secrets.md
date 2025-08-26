# Uso de Secret Scope `scope-dev` y secrets en Azure Databricks

## 📌 Descripción general
Este documento describe cómo los notebooks consumen credenciales de **Azure Key Vault** a través del **Secret Scope** `scope-dev` en Azure Databricks, evitando exponer credenciales en código y permitiendo la rotación centralizada vía RBAC.

---

## ✅ Requisitos previos (alto nivel)
- **Azure Key Vault** con los secretos creados.
- **Azure RBAC** habilitado en el Key Vault.
- La **Managed Identity** (UAMI/SAMI) del workspace con rol **Key Vault Secrets User** sobre el Key Vault.
- **Secret Scope** `scope-dev` en Databricks apuntando a ese Key Vault.

> Nota: Los nombres de las *properties* y *actions* van en **English** (ej. `scope`, `key`, `get`), las explicaciones en español.

---

## 🔑 Scope y secrets utilizados

**Scope:** `scope-dev`

**Secrets (keys):**
- `secret-sql-ventas-url` → URL JDBC de la BD Ventas  
- `secret-sql-ventas-user` → Usuario de la BD Ventas  
- `secret-sql-ventas-password` → Password del usuario de la BD Ventas  
- `secret-sql-clinica-url` → URL JDBC de la BD Clinica  
- `secret-sql-clinica-user` → Usuario de la BD Clinica  
- `secret-sql-clinica-password` → Password del usuario de la BD Clinica  

---

## 🧪 Verificación rápida

Listar scopes:
```python
dbutils.secrets.listScopes()
```

Listar keys dentro del scope:
```python
dbutils.secrets.list("scope-dev")
```

> Los valores de los secrets **no** se imprimen por seguridad; solo verás los nombres de las keys.

---

## ⚙️ Consumo en notebooks

### 1) Read secrets
```python
url_conexion = dbutils.secrets.get("scope-dev", "secret-sql-ventas-url")
usuario      = dbutils.secrets.get("scope-dev", "secret-sql-ventas-user")
password     = dbutils.secrets.get("scope-dev", "secret-sql-ventas-password")
```

```python
url_conexion = dbutils.secrets.get("scope-dev", "secret-sql-clinica-url")
usuario      = dbutils.secrets.get("scope-dev", "secret-sql-clinica-user")
password     = dbutils.secrets.get("scope-dev", "secret-sql-clinica-password")
```

### 2) Use en conexión JDBC (Spark)
```python
df = (spark.read.format("jdbc")
      .option("url", url_conexion)
      .option("user", usuario)
      .option("password", password)
      .option("dbtable", "dbo.FactVentas")
      .load())
display(df)
```

### 3) Write (opcional)
```python
(df.write.mode("append")
   .format("jdbc")
   .option("url", url_conexion)
   .option("user", usuario)
   .option("password", password)
   .option("dbtable", "dbo.LogIngesta")
   .save())
```

---

## 🔒 Buenas prácticas
- **No** imprimas ni loguees variables que contengan secretos.
- Usa **grupos de AAD** (no usuarios individuales) para asignar acceso en Key Vault.
- Separa scopes por entorno: `scope-dev`, `scope-qa`, `scope-prd`.
- Versiona en Git solo el **nombre** del scope/keys; jamás exportes valores de secretos.

---

## 🛠️ Troubleshooting
- **`PERMISSION_DENIED: Failed to read secret from scope`**  
  - Verifica que el scope existe: `dbutils.secrets.listScopes()`  
  - Asegura que la Managed Identity tenga **Key Vault Secrets User** en el KV.  
  - Chequea que el scope esté apuntando al **Key Vault correcto** (DNS/Resource ID).

- **`KeyError: Secret not found`**  
  - Asegura que la key existe en KV y coincide exactamente:  
    `secret-sql-ventas-url`, `secret-sql-ventas-user`, `secret-sql-ventas-password`.

- **Rotación de credenciales**  
  - Rota en **Key Vault** y mantén el mismo nombre de key; los notebooks no requieren cambios.

---

## 📚 Referencias
- Azure Databricks – Secret Scopes  
- Azure Databricks – Azure Key Vault–backed scopes  
- Azure Key Vault – RBAC Guide
