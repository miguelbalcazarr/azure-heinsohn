# Uso de Storage Credentials y External Locations en Unity Catalog

## 📌 Descripción general
Cuando se habilita **Unity Catalog (UC)** en Azure Databricks, se define un **storage location por defecto** asociado al metastore, respaldado por un contenedor en ADLS Gen2.  
Este almacenamiento sirve como ubicación principal para catálogos, esquemas y tablas **gestionadas**.

Sin embargo, en muchos escenarios se requiere trabajar con **otros data lakes** o **contenedores adicionales** (por ejemplo, un ADLS independiente para entornos de desarrollo, QA o producción).  
En estos casos se utilizan dos elementos clave:

- **Storage Credentials**  
- **External Locations**

---

## 🗂️ Storage Credentials
Un **Storage Credential** representa la configuración de acceso seguro hacia una cuenta de almacenamiento en Azure.  
Permite a Unity Catalog autenticar y autorizar el acceso a un ADLS Gen2 mediante una **Managed Identity (UAMI o SAMI)** previamente configurada.

### Beneficios
- **Seguridad centralizada:** el acceso se gestiona desde Unity Catalog, evitando exponer credenciales sensibles en código.  
- **Flexibilidad:** un mismo workspace puede acceder a múltiples cuentas de almacenamiento mediante diferentes credenciales.  
- **Separación de entornos:** permite aislar físicamente los datos de catálogos como *desarrollo*, *testing* o *producción*.  
- **Escalabilidad:** facilita la incorporación de nuevos data lakes o buckets en la misma gobernanza.

### Ejemplo en SQL
```sql
CREATE STORAGE CREDENTIAL cred_dev
WITH IDENTITY 'uami-dev-identity'
COMMENT 'Credencial para acceder a ADLS del catálogo Desarrollo';
```

---

## 🌐 External Locations
Un **External Location** es una ruta de almacenamiento en ADLS Gen2, asociada a un **Storage Credential**, que se registra en Unity Catalog.  
Esto permite a los usuarios crear tablas **externas** en esa ruta bajo control de UC.

### Beneficios
- **Gobernanza unificada:** UC administra permisos de acceso a rutas externas con el mismo modelo de privilegios que usa para catálogos y tablas.  
- **Control de acceso granular:** se pueden asignar privilegios de lectura/escritura a grupos o usuarios específicos en cada ruta externa.  
- **Flexibilidad en la organización de datos:** diferentes external locations para distintos catálogos o capas (*bronze, silver, gold*).  
- **Compatibilidad con entornos aislados:** útil cuando el equipo quiere separar datos de *desarrollo* en un ADLS distinto del metastore por defecto.

### Ejemplo en SQL
```sql
CREATE EXTERNAL LOCATION loc_dev
URL 'abfss://dev-container@storagedevaccount.dfs.core.windows.net/'
WITH STORAGE CREDENTIAL cred_dev
COMMENT 'Ubicación externa para datos del catálogo Desarrollo';
```

---

## 🚀 Caso de uso: Catálogo Desarrollo
1. Se habilitó Unity Catalog con un **storage location por defecto** para el metastore.  
2. Se desplegó un **nuevo ADLS** dedicado a datos de desarrollo.  
3. Para integrarlo en UC:
   - Se creó un **Storage Credential** (`cred_dev`) asociado a la **UAMI** con permisos en el nuevo ADLS.  
   - Se definió un **External Location** (`loc_dev`) apuntando al contenedor del nuevo ADLS.  
4. Al crear el catálogo *desarrollo*, se referencia el external location:  

```sql
CREATE CATALOG desarrollo
MANAGED LOCATION 'abfss://dev-container@storagedevaccount.dfs.core.windows.net/'
COMMENT 'Catálogo de desarrollo con datos aislados en ADLS propio';
```

---

## 🎯 Buenas prácticas
- Usar **UAMI** dedicadas para cada Storage Credential (por entorno o dominio de datos).  
- Organizar los External Locations siguiendo las capas del Lakehouse (ej. `bronze_dev`, `silver_dev`, `gold_dev`).  
- Asignar privilegios solo a **grupos de AAD** en los external locations, nunca a usuarios individuales.  
- Versionar los scripts SQL de creación de credenciales y external locations en **Git**.  

---

## 📚 Referencias
- [Unity Catalog: Storage Credentials](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/storage-credentials)  
- [Unity Catalog: External Locations](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/external-locations)  
