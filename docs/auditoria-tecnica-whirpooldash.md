# Auditoría técnica del proyecto whirpooldash

**Fecha de auditoría:** 2026-06-19 (Fase 1) · **Ampliación Fase 2:** 2026-06-19  
**Ruta del repositorio:** `C:\Users\naran\OneDrive - Instituto Tecnologico y de Estudios Superiores de Monterrey\IMD 7\whirpooldash`  
**Commit analizado:** `f1658d9` — *feat: update dashboard styling and add new partner*

---

## 1. Resumen ejecutivo

**whirpooldash** es un dashboard analítico interno orientado al seguimiento comercial de electrodomésticos Whirlpool. La aplicación combina **Streamlit** como capa de presentación, **FastAPI** como servicio auxiliar de predicción legacy, **PostgreSQL en Azure** como fuente operativa de negocio y **Azure Blob Storage** como repositorio de artefactos de machine learning serializados en formato `.pkl`. El repositorio conserva la lógica de integración, la estructura de consultas SQL y las rutas de despliegue, pero no incluye los datos transaccionales ni los modelos entrenados que hacían operativa la versión original.

Desde una perspectiva de viabilidad técnica, el proyecto **sí constituye una base sólida** para continuar desarrollo, evaluación académica o reconstrucción controlada. No obstante, **no es plenamente reproducible** con el estado actual del repositorio porque faltan tres clases de activos externos: (a) tablas `sellout` e `iqsigma`, (b) blobs `.pkl` con SAS vigentes o copias locales, y (c) secretos de despliegue asociados a la infraestructura del autor original. La ruta más segura consiste en convertir el sistema en una **versión local reproducible** —mediante CSV, SQLite o PostgreSQL local y modelos en disco— y, solo después, reconectar opcionalmente a un **Azure propio** bajo tu control.

### ¿Está funcional hoy?

| Área | Estado |
|------|--------|
| UI Streamlit (arranque general) | Parcial — arranca tras instalar dependencias |
| Dashboard Performance (KPIs) | Depende de PostgreSQL Azure — **falla sin credenciales válidas** |
| Dashboard Performance (gráficos) | **Funciona** — datos estáticos hardcodeados en `components/dashboard.py` |
| Market Performance | Depende de PostgreSQL — **sin datos locales alternativos** |
| Prediction (XGBoost) | Depende de blobs Azure — **tokens SAS expirados (HTTP 403)** |
| Price Calculator (FastAPI + Azure LSTM) | Backend importable; UI **no expuesta** en navegación actual |
| CI/CD y despliegue | Workflow presente; **no debe reutilizarse** sin secretos e infraestructura propios |

### Conclusión de viabilidad (síntesis Fase 2)

El análisis funcional profundo (Sección 10) permite concluir lo siguiente. Primero, PostgreSQL Azure no es un accesorio: es el **sistema nervioso de datos** que alimenta KPIs de sellout, inteligencia de mercado y metadatos de partners/SKUs. Segundo, los modelos `.pkl` en Blob Storage son **artefactos de inferencia**, no meramente archivos auxiliares; la pestaña Prediction depende de tres pickles coordinados (modelo, columnas codificadas y dataset fuente). Tercero, los SAS tokens embebidos representan un riesgo de seguridad y un punto único de fallo temporal: al expirar, la descarga remota deja de funcionar aunque el código permanezca intacto. Cuarto, el workflow de GitHub Actions está diseñado para la cuenta Docker Hub y el App Service del autor; **no debe reutilizarse** su infraestructura ni sus secretos sin autorización explícita.

Una versión aproximada **sí puede funcionar** sin Azure original mediante CSV locales (`sample_sellout.csv`, `sample_iqsigma.csv`), un `DataProvider` unificado, modelos en `models/` ignorados por Git y fallback por reglas cuando falte ML. Esa ruta preserva la demostrabilidad del proyecto aunque no reproduzca fidelidad numérica plena respecto al entorno de producción original.

---

## 2. Estado del repositorio

### 2.1 Ubicación y raíz Git

| Comando | Resultado |
|---------|-----------|
| `pwd` | `...\IMD 7\whirpooldash` |
| `git rev-parse --show-toplevel` | `C:/Users/naran/.../IMD 7/whirpooldash` |

### 2.2 Rama, estado y último commit

| Elemento | Valor |
|----------|-------|
| Rama actual | `main` |
| Estado de trabajo | Limpio (`nothing to commit, working tree clean`) |
| HEAD | `f1658d9` |
| Autor último commit | Beltry201 |
| Fecha | 2025-12-04 |
| Mensaje | `feat: update dashboard styling and add new partner` |

### 2.3 Remotos

| Remoto | Fetch | Push | Observación |
|--------|-------|------|-------------|
| `origin` | `https://github.com/narajoEmmanuel/whirpooldash.git` | mismo URL | **Tu repo principal** |
| `upstream` | `https://github.com/joelvshimself/whirpooldash.git` | `DISABLED` | Solo lectura; push bloqueado |

**Confirmación:** `origin` apunta correctamente a tu fork (`narajoEmmanuel/whirpooldash`). `upstream` tiene push deshabilitado (`remote.upstream.pushurl = DISABLED`), lo que protege contra pushes accidentales al repo original.

### 2.4 Sincronización entre remotos

```
origin/main     → f1658d9
upstream/main   → f1658d9
HEAD (main)     → f1658d9
```

Los tres apuntan al **mismo commit**. Tu `main` está al día con `origin/main` y coincide con `upstream/main`.

### 2.5 Ramas remotas

**En `origin` (tu repo):**
- Solo `origin/main`

**En `upstream` (repo original — referencia):**
- `upstream/main`
- `upstream/3tabs`
- `upstream/azuret`
- `upstream/david/backend`
- `upstream/english`
- `upstream/filtros-skus`
- `upstream/interface`
- `upstream/sellinview`
- `upstream/selloutview`
- `upstream/skus-cat`
- `upstream/ui`

Las ramas adicionales existen **solo en upstream** como referencia histórica de desarrollo. No están en tu `origin`.

### 2.6 Historial reciente (20 commits)

```
f1658d9 (HEAD -> main, origin/main, upstream/main) feat: update dashboard styling and add new partner
758c73f feat: add price statement rendering to price calculator
5d160cb test: preload data.
73f6098 fix: TP drop down options for price calculator
7adaed8 lil
8e91ad9 Refactor prediction functionality in run_model and dashboard components
14ac930 Merge pull request #4 from joelvshimself/english
f465c58 (upstream/english) labels
cae83f6 Merge pull request #3 from joelvshimself/skus-cat
17d8280 (upstream/skus-cat) dont show unknow
726a45b Merge pull request #2 from joelvshimself/interface
e4f2f16 (upstream/interface) sellout kpis
d88f6bd Merge pull request #1 from joelvshimself/filtros-skus
6940b75 (upstream/filtros-skus) add timeline
98faf0e add valid skus
d2e57a7 Enhance logging in XGBoost prediction service
86938bd Add XGBoost model integration and update prediction dashboard
6fc4e1a (upstream/sellinview) Merge remote-tracking branch 'origin/main' into sellinview
016bbd9 Merge remote-tracking branch 'origin/selloutview' into sellinview
b751f01 Add Market Performance Component and Update Navigation
```

---

## 3. Arquitectura general

### 3.1 Tipo de aplicación

Dashboard analítico B2B para electrodomésticos Whirlpool con:
- KPIs de sellout por trading partner
- Análisis de participación de mercado por marca/categoría
- Predicción de precios con modelos ML (XGBoost en UI principal; LSTM/Azure en backend legacy)

### 3.2 Framework principal

| Capa | Tecnología |
|------|------------|
| Frontend | **Streamlit** (`app.py`) |
| Backend API | **FastAPI** + **Uvicorn** (`backend.py`, puerto 8000) |
| Visualización | **Plotly** |
| Datos | **SQLAlchemy** + **psycopg2** → PostgreSQL |
| ML | **XGBoost** (predicción activa), **TensorFlow/LSTM** (opcional/fallback) |
| Contenedor | **Docker** (Python 3.11-slim) |

### 3.3 Rol de archivos principales

| Archivo | Rol |
|---------|-----|
| `app.py` | Punto de entrada Streamlit. Inicia FastAPI en hilo daemon, precarga datos, define sidebar con 3 pestañas y CSS custom. |
| `backend.py` | API REST FastAPI: `/api/predict`, `/api/history`, `/api/partners`, `/api/reload-model`, `/api/train`. Usa `AzureModelService` + fallback LSTM local. |
| `config.py` | Variables de entorno, defaults, carga de SKUs desde `unique_skus.txt`, consultas a BD para categorías y partners. |
| `requirements.txt` | Dependencias Python con rangos mínimos (sin pins estrictos). |
| `Dockerfile` | Imagen Python 3.11-slim, usuario no-root, expone puerto 8080. |
| `entrypoint.sh` | Ejecuta `streamlit run app.py` en `$PORT` (default 8080). |
| `README.md` | Documentación básica de setup y estructura. |
| `README_DEPLOY.md` | Guía de despliegue Docker Hub + Azure App Service + secretos GitHub. |
| `INSTALL.md` | Notas de instalación (PyPy/TensorFlow); algo desactualizado vs código actual. |
| `unique_skus.txt` | 468 SKUs reales (uno por línea) usados en selectores y predicción. |
| `azure_creds.json` | Placeholder para Service Principal de Azure CI/CD; **archivo vacío (0 bytes)** versionado en Git. |

### 3.4 Carpetas

| Carpeta | Contenido y rol |
|---------|----------------|
| `components/` | UI Streamlit: `dashboard.py` (sellout + predicción XGBoost), `market_performance.py`, `price_calculator.py`, `sku_table.py`. |
| `services/` | Lógica de negocio: BD (`db.py`), KPIs (`sellout_kpis.py`), mercado (`market_performance.py`), Azure models (`azure_model_service.py`), XGBoost remoto (`run_model.py`), API client (`api_client.py`). |
| `data/` | Abstracción `DataSource` + `MockDataSource` (mock para FastAPI/LSTM; **no usado** por KPIs principales). |
| `ml/` | `lstm_model.py`, `data_processor.py` — LSTM con fallback sin TensorFlow. |
| `utils/` | Formateo de moneda, porcentajes, nús. |
| `assets/` | `whirpool_logo.png` (~59 KB) para sidebar. |
| `.github/workflows/` | `deploy.yml` — CI/CD a Docker Hub + Azure Web App. |

**Nota:** No hay carpeta `models/` en el repo (ignorada por `.gitignore`). Los modelos viven en Azure Blob.

### 3.5 Flujo general de datos

```mermaid
flowchart TB
    subgraph UI["Streamlit app.py"]
        Tab1[Dashboard Performance]
        Tab2[Market Performance]
        Tab3[Prediction XGBoost]
    end

    subgraph Backend["FastAPI backend.py :8000"]
        PredictAPI["/api/predict"]
        AzureSvc[AzureModelService]
        LSTM[LSTM fallback]
    end

    subgraph External["Servicios externos"]
        PG[(Azure PostgreSQL<br/>sellout, iqsigma)]
        Blob[(Azure Blob Storage<br/>modelstoragest)]
    end

    subgraph Local["Datos locales"]
        SKUs[unique_skus.txt]
        Mock[MockDataSource]
        Static[Gráficos estáticos<br/>en dashboard.py]
    end

    Tab1 -->|KPIs| PG
    Tab1 -->|Charts| Static
    Tab2 --> PG
    Tab3 -->|download pickles| Blob
    Tab3 --> SKUs

    app.py -->|thread daemon| Backend
    PredictAPI --> AzureSvc --> Blob
    PredictAPI --> Mock
    AzureSvc -->|fallback| LSTM
```

### 3.6 Clasificación por capa

| Capa | Archivos / componentes |
|------|----------------------|
| Frontend | `app.py`, `components/*` |
| Backend | `backend.py`, `services/api_client.py` |
| Cálculo / ML | `services/run_model.py`, `services/azure_model_service.py`, `ml/*` |
| Datos | `services/db.py`, `services/sellout_kpis.py`, `services/market_performance.py`, `data/*`, `unique_skus.txt` |
| Configuración | `config.py`, variables de entorno, `.env` (no presente localmente) |
| Despliegue | `Dockerfile`, `entrypoint.sh`, `.github/workflows/deploy.yml`, `README_DEPLOY.md` |

### 3.7 Navegación actual vs código legacy

`app.py` importa `render_price_calculator` y `render_sku_table` pero **no los renderiza** en ninguna pestaña. La UI activa tiene solo:

1. **Dashboard Performance** — KPIs de BD + gráficos estáticos
2. **Market Performance** — consultas a `iqsigma`
3. **Prediction** — XGBoost vía `generate_price_prediction_statement()`

El Price Calculator con FastAPI/Azure LSTM quedó como código disponible pero no integrado en la navegación principal.

---

## 4. Dependencias y entorno

### 4.1 `requirements.txt` (dependencias declaradas)

```
streamlit>=1.28.0
pandas>=2.0.0
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0
plotly>=5.17.0
requests>=2.31.0
python-dotenv>=1.0.0
fastapi>=0.104.0
uvicorn>=0.24.0
numpy>=1.24.0
xgboost>=2.0.0
starlette>=0.40.0,<0.50.0
pydantic>=1.7.4,<3.0.0
annotated-doc>=0.0.2
protobuf>=3.20.0
streamlit-custom-sidebar>=0.0.16
streamlit-float>=0.1.0
```

Comentadas como opcionales: `tensorflow`, `scikit-learn`, `altair`, `pyarrow`, `pydeck`.

### 4.2 Uso probable por dependencia

| Dependencia | Uso |
|-------------|-----|
| `streamlit` | UI completa |
| `pandas` | DataFrames, SQL results, predicción XGBoost |
| `sqlalchemy` + `psycopg2-binary` | Conexión PostgreSQL Azure |
| `plotly` | Gráficos interactivos |
| `requests` | Descarga de blobs Azure, cliente HTTP al backend |
| `python-dotenv` | Carga `.env` en `config.py` |
| `fastapi` + `uvicorn` + `starlette` + `pydantic` | Backend embebido |
| `numpy` | Procesamiento numérico, fallback LSTM |
| `xgboost` | Deserializar y ejecutar modelos `.pkl` remotos |
| `protobuf` | Dependencia transitiva (XGBoost/ML stack) |
| `streamlit-custom-sidebar`, `streamlit-float` | Declaradas pero **no importadas** en código revisado |

### 4.3 Agrupación temática

| Categoría | Paquetes |
|----------|----------|
| Azure | Ningún SDK oficial (`azure-storage-blob`, etc.). Solo URLs HTTP + SAS tokens. |
| APIs web | `requests`, `fastapi`, `uvicorn` |
| Visualización | `plotly`, `streamlit` |
| Machine learning | `xgboost` (activo), `tensorflow`/`scikit-learn` (opcionales, no instalados) |
| Autenticación/credenciales | `python-dotenv`; credenciales en `config.py` y blobs |
| Despliegue | Docker base Python; GitHub Actions usa `azure/login`, `azure/cli`, `docker/*` |

### 4.4 Riesgos de instalación

| Riesgo | Detalle |
|-------|---------|
| Sin versiones pinneadas | Builds no reproducibles; posibles conflictos futuros |
| Python 3.13 local | Proyecto Docker usa 3.11; XGBoost/psycopg2 funcionaron en 3.13 en prueba, pero no garantizado |
| TensorFlow ausente | LSTM usa fallback trend-based; predicciones menos precisas |
| `streamlit-custom-sidebar` / `streamlit-float` | Instaladas pero aparentemente no usadas — peso extra |
| Entorno sin venv en repo | No hay `.venv` versionado; conviene crear uno local |

### 4.5 Estado de imports (prueba 2026-06-19)

Antes de instalar faltaban: `sqlalchemy`, `psycopg2`, `dotenv`, `fastapi`, `uvicorn`, `xgboost`.  
Tras `pip install` de esos paquetes, **todos los imports principales pasaron**.

---

## 5. Ejecución local

### 5.1 Requisitos

- Python 3.11+ recomendado (Docker usa 3.11)
- PowerShell en Windows
- Acceso de red si se usan BD/Azure (opcional para UI parcial)

### 5.2 Instalación (PowerShell)

```powershell
cd "C:\Users\naran\OneDrive - Instituto Tecnologico y de Estudios Superiores de Monterrey\IMD 7\whirpooldash"

# Opcional pero recomendado
python -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 5.3 Ejecutar Streamlit

```powershell
cd "C:\Users\naran\OneDrive - Instituto Tecnologico y de Estudios Superiores de Monterrey\IMD 7\whirpooldash"
$env:DATA_SOURCE_TYPE = "mock"   # default; no afecta KPIs principales
streamlit run app.py
```

- UI: `http://localhost:8501` (puerto default Streamlit)
- Backend FastAPI: `http://localhost:8000` (arranca automáticamente en hilo daemon)
- Docs API: `http://localhost:8000/docs`

### 5.4 Ejecutar FastAPI standalone (opcional)

```powershell
python backend.py
# o
uvicorn backend:app --host 0.0.0.0 --port 8000 --reload
```

### 5.5 Docker (no verificado en esta máquina)

Docker **no está instalado** en el entorno auditado (`docker` no reconocido). Comandos documentados:

```powershell
docker build -t whirpooldash:local .
docker run --rm -it -p 8080:8080 -e PORT=8080 whirpooldash:local
# Abrir http://localhost:8080
```

### 5.6 Variables de entorno relevantes

| Variable | Default en código | Necesaria para |
|----------|-------------------|---------------|
| `DATA_SOURCE_TYPE` | `mock` | Capa mock del FastAPI (no KPIs principales) |
| `POSTGRES_CONNECTION_STRING` | Host Azure + user/pass placeholder | KPIs, Market Performance, categorías SKU |
| `API_BASE_URL` | `http://localhost:8000` | Price Calculator API client |
| `API_PORT` | `8000` | Backend embebido |
| `AZURE_BLOB_BASE_URL` | `https://modelstoragest.blob.core.windows.net/models` | Modelos LSTM por partner |
| `AZURE_BLOB_SAS_TOKEN` | Token SAS embebido en `config.py` | Acceso lectura blobs |
| `PORT` | `8080` (Docker/Azure) | Contenedor Streamlit |

**No existe `.env` local.** Crear uno es recomendable para no depender de defaults embebidos.

### 5.7 Errores probables al correr

| Error | Causa probable |
|-------|----------------|
| `ModuleNotFoundError: dotenv` / `sqlalchemy` / `xgboost` | Dependencias no instaladas |
| KPIs en 0 / warnings BD | PostgreSQL rechaza credenciales placeholder |
| `Prediction failed: 403` | SAS tokens de `run_model.py` expirados |
| `TensorFlow not available` | Normal; usa fallback (warning, no fatal) |
| Puerto 8000 ocupado | Backend previo no liberado |

### 5.8 Pruebas seguras realizadas

| Prueba | Resultado |
|--------|-----------|
| Import `backend` + listado de rutas | OK (`/`, `/api/predict`, etc.) |
| Import `MockDataSource` | OK |
| Import `run_model` | OK |
| Conexión PostgreSQL | **FAIL** — `password authentication failed` (host responde) |
| HEAD blob partner model | **404** — archivo no encontrado o nombre incorrecto |
| HEAD blob XGB final model | **403** — SAS expirado |

---

## 6. Datos disponibles

### 6.1 En el repositorio

| Recurso | Ubicación | Contenido |
|---------|-----------|-----------|
| Lista de SKUs | `unique_skus.txt` | **468 SKUs** (códigos reales Whirlpool, ej. `7KFCB519MPA`) |
| Logo | `assets/whirpool_logo.png` | Branding sidebar |
| Mock datasets | `data/mock_data_source.py` | KPIs/charts ficticios para capa FastAPI |
| Gráficos dashboard | `components/dashboard.py` | Series estáticas de retailers (Coppel, Elektra, etc.) |
| Métricas evaluación modelo | `components/dashboard.py` | Valores inventados (R²=0.94, etc.) |

### 6.2 Datos **no** presentes localmente

| Dato | Tabla / origen | Usado en |
|------|---------------|----------|
| Sellout histórico | PostgreSQL `sellout` | KPIs Dashboard Performance |
| Datos mercado | PostgreSQL `iqsigma` | Market Performance, categorías SKU |
| Modelos XGBoost | Azure Blob `final_xgb_*.pkl` | Pestaña Prediction |
| Modelos LSTM por partner | Azure Blob `{partner}_model.pkl` | FastAPI `/api/predict` |
| Dataset fuente XGBoost | Azure Blob `final_xgb_source_data.pkl` | Predicción |

### 6.3 Esquema inferido de tablas PostgreSQL

**Tabla `sellout`** (columnas referenciadas):
- `DATE`, `QTY`, `Real_price`, `GROSS_SALES`, `TP` (trading partner)

**Tabla `iqsigma`** (columnas referenciadas):
- `DATE`, `BRAND`, `PRICE_SOLD`, `CATEGORY`, `SKU`

### 6.4 ¿Se pueden reconstruir datos faltantes?

| Fuente | ¿Recuperable del repo? |
|--------|---------------------|
| SKUs válidos | **Sí** — `unique_skus.txt` |
| Partners default | **Parcial** — lista hardcodeada en `config.py` si BD falla |
| Categorías por SKU | **No** — requiere `iqsigma` o export CSV externo |
| Histórico sellout/iqsigma | **No** — datos privados en Azure PostgreSQL |
| Modelos ML | **No en repo** — solo URLs con SAS en código (varios expirados) |

### 6.5 Trabajo parcial sin datos privados

- Explorar UI y gráficos estáticos del Dashboard Performance
- Probar navegación, CSS, logo, selectores de SKU (468 opciones)
- Desarrollar/refactorizar componentes con mocks
- Ejecutar FastAPI con `MockDataSource` (predicciones sintéticas)
- **No** predicciones XGBoost reales ni KPIs de mercado sin BD/blobs

---

## 7. Conexiones externas y web

### 7.1 Conexiones identificadas

| Destino | Protocolo | Archivo(s) | Activa | Depende de config |
|---------|-----------|------------|--------|-------------------|
| PostgreSQL Azure | TCP/SSL | `config.py`, `services/db.py` | Sí (intento al cargar) | `POSTGRES_CONNECTION_STRING` |
| Azure Blob modelos LSTM | HTTPS GET | `services/azure_model_service.py` | On-demand | `AZURE_BLOB_*` |
| Azure Blob XGBoost | HTTPS GET | `services/run_model.py` | On-demand | URLs hardcodeadas + SAS |
| FastAPI local | HTTP | `services/api_client.py` → `localhost:8000` | Sí (si Streamlit corre) | `API_BASE_URL` |
| Azure Web App (prod) | HTTPS | `.github/workflows/deploy.yml` | Solo CI/CD | Secretos GitHub |
| Docker Hub | HTTPS | `.github/workflows/deploy.yml` | Solo CI/CD | `DOCKERHUB_*` |

### 7.2 Endpoints FastAPI (`backend.py`)

| Método | Ruta | Función |
|--------|------|---------|
| GET | `/` | Health check |
| POST | `/api/predict` | Predicción precio (Azure LSTM → fallback local) |
| GET | `/api/history` | Historial predicciones |
| GET | `/api/partners` | Lista partners |
| POST | `/api/reload-model` | Recarga modelo Azure |
| POST | `/api/train` | Reentrenar LSTM local |

### 7.3 URLs Azure Blob (sin secretos)

| Recurso | URL base |
|---------|----------|
| Storage account | `modelstoragest.blob.core.windows.net` |
| Contenedor modelos LSTM | `/models/` |
| Contenedor datos XGB | `/data/` |
| App Service (placeholder CI) | `streamlit-app-demos-*.canadacentral-01.azurewebsites.net` |

### 7.4 `requests` en el código

- `services/run_model.py` — descarga pickles XGBoost
- `services/azure_model_service.py` — descarga modelos LSTM por partner
- `services/api_client.py` — POST/GET al backend local

No se encontró uso de `httpx` ni `fetch` (frontend Python).

### 7.5 Datos obtenibles externamente (con acceso)

| Servicio | Qué devuelve | Requisito |
|---------|--------------|-----------|
| PostgreSQL | KPIs sellout, stats iqsigma, partners, categorías SKU | Connection string válida |
| Blob `/models/*.pkl` | Modelos LSTM por trading partner | SAS token válido |
| Blob `/data/final_xgb_*.pkl` | Modelo XGB, columnas encoded, dataset fuente | SAS token válido (renovar) |

---

## 8. Azure y credenciales

### 8.1 Servicios Azure involucrados

| Servicio | Uso en proyecto |
|---------|-----------------|
| **Azure Database for PostgreSQL** | Host `streamlit-postgres.postgres.database.azure.com` — tablas `sellout`, `iqsigma` |
| **Azure Blob Storage** | Cuenta `modelstoragest` — modelos `.pkl` y datasets |
| **Azure App Service** | Web App `streamlit-app-demos`, resource group `st`, región Canada Central |
| **Azure AD Service Principal** | CI/CD vía `AZURE_CREDENTIALS` en GitHub Actions |

**No aparecen:** Cosmos DB, Key Vault, Container Registry propio (usa Docker Hub), `DefaultAzureCredential`, connection strings clásicas de Storage Account.

### 8.2 `azure_creds.json`

| Aspecto | Hallazgo |
|---------|---------|
| Existe localmente | Sí |
| Tamaño | **0 bytes (vacío)** |
| Versionado en Git | Sí (commit `f609a19` "CI&CD") |
| Contenido en historial | Commit creó archivo vacío; **no se encontraron secretos en historial Git** |
| Uso esperado | JSON de Service Principal para `azure/login@v2` en GitHub Actions |

### 8.3 Configuración Azure por canal

| Canal | Qué configura |
|-------|---------------|
| `config.py` | `POSTGRES_CONNECTION_STRING`, `AZURE_BLOB_BASE_URL`, `AZURE_BLOB_SAS_TOKEN` (defaults embebidos) |
| Variables de entorno | Override de lo anterior vía `.env` / App Settings |
| `services/run_model.py` | URLs blob XGBoost con SAS **hardcodeadas** (independiente de env) |
| `Dockerfile` / `entrypoint.sh` | Puerto 8080 para App Service |
| `.github/workflows/deploy.yml` | Login Azure, deploy imagen Docker Hub → Web App |
| `README_DEPLOY.md` | Documenta SP, secretos, comandos `az` |

### 8.4 Qué parte de Azure es necesaria

| Funcionalidad | Azure requerido |
|--------------|----------------|
| KPIs Dashboard / Market | **PostgreSQL** obligatorio |
| Predicción XGBoost (UI) | **Blob Storage** obligatorio |
| Price Calculator LSTM | **Blob Storage** (modelos por partner) |
| Despliegue producción | **App Service** + Docker Hub + Service Principal |

### 8.5 Recuperación de configuración

| Fuente | Qué se puede recuperar | Qué no |
|--------|------------------------|--------|
| Archivos actuales | Host PG, host blob, nombres Web App/RG, estructura URLs | Password PG real, SAS vigentes a largo plazo |
| Historial Git | Cuándo se añadieron integraciones (commits `b751f01`, `c1777be`, `86938bd`, `f609a19`) | Secretos completos (parcialmente redactados en historial) |
| README_DEPLOY.md | Nombres recursos, flujo CI/CD, comandos SP | Valores de secretos |
| Workflows | `WEBAPP_NAME=streamlit-app-demos`, `RESOURCE_GROUP=st`, imagen `joecast208/whirpooldash` | Credenciales (viven en GitHub Secrets) |
| Variables esperadas | Lista completa de env vars | Valores |

**Pedir al dueño original (`joelvshimself`):**
- Connection string PostgreSQL con usuario/contraseña válidos (o export de tablas)
- Renovación de SAS tokens para contenedores `models` y `data`
- Acceso al resource group `st` o nuevo entorno Azure
- Confirmación de si Docker Hub `joecast208/whirpooldash` sigue activo

### 8.6 Secretos detectados en código (REDACATOS)

> **No copies estos valores del código fuente a documentos públicos. Rota si fueron expuestos.**

| Archivo | Líneas aprox. | Tipo | Riesgo | Acción recomendada |
|---------|---------------|------|--------|-------------------|
| `config.py` | 14-16 | Connection string PostgreSQL con password placeholder | Medio — host expuesto; password default no funciona pero patrón visible | Mover a variable de entorno; rotar password en Azure |
| `config.py` | 28-30 | SAS token contenedor Azure Blob | **Alto** — token lectura embebido en repo | Revocar SAS, usar env var / Key Vault, limpiar historial Git |
| `services/run_model.py` | 14-27 | 3 URLs blob + SAS tokens | **Alto** — tokens expirados pero aún en historial | Externalizar a env; revocar tokens; descargar modelos a storage privado |
| `.github/workflows/deploy.yml` | — | Referencia a `secrets.AZURE_CREDENTIALS` | Bajo (solo referencia) | Configurar en tu fork con tu SP |

### 8.7 ¿Trabajar sin Azure?

| Modo | Viabilidad |
|------|------------|
| Sin Azure | UI parcial + FastAPI mock; sin KPIs reales ni predicción XGBoost |
| Azure simulado | Mock de `run_query` + pickles locales descargados manualmente — requiere desarrollo |
| Datos locales | Export CSV/Parquet de tablas + modelos `.pkl` en disco — **mejor opción offline** |
| Azure real | Necesario para paridad con producción |

---

## 9. Recuperación de información

### 9.1 Recuperable desde el repo

- Código fuente completo y arquitectura
- Lista de 468 SKUs
- Nombres de recursos Azure (host PG, storage account, web app, resource group)
- Workflow CI/CD y documentación de despliegue
- Defaults de partners, regiones, estructura SQL
- Ramas históricas en `upstream/*` para comparar features

### 9.2 No recuperable solo desde el repo

- Contenido de tablas `sellout` e `iqsigma`
- Modelos `.pkl` (ignorados por git; solo remotos)
- Secretos de GitHub Actions del repo original
- SAS tokens vigentes (los embebidos están expirados o expuestos)

### 9.3 Depende de accesos externos

- Credenciales PostgreSQL del equipo original
- Cuenta/storage Azure `modelstoragest`
- Service Principal o publish profile para App Service `streamlit-app-demos`
- Repositorio Docker Hub `joecast208/whirpooldash`

La Sección 10 desarrolla el análisis funcional de estas dependencias y propone rutas de reemplazo.

---

## 10. Análisis funcional profundo de las integraciones críticas

Esta sección corresponde a la **Fase 2** de auditoría. Su objetivo es interpretar, a partir del código fuente y la documentación interna del repositorio, el **propósito de diseño** de cada integración externa, las **brechas de información** que impiden la reproducibilidad plena y las **alternativas técnicas** para operar el proyecto sin acceso al Azure original. Las afirmaciones se sustentan exclusivamente en artefactos del repositorio (`config.py`, `services/*`, `components/*`, `.github/workflows/deploy.yml`, entre otros); no se invocan fuentes externas inventadas.

### 10.1 PostgreSQL Azure como fuente operativa del dashboard

#### Rol arquitectónico

PostgreSQL en Azure cumple la función de **fuente operativa de verdad** (*source of truth*) para las métricas de negocio que el dashboard presenta como datos “reales”. A diferencia de la capa `MockDataSource` definida en `data/mock_data_source.py`, que fue diseñada para abstraer datos ficticios hacia el backend FastAPI/LSTM, las pantallas principales de Streamlit **evitan esa abstracción** y consultan PostgreSQL de forma directa mediante `services/db.py` → `run_query()`.

La cadena de acceso se configura en `config.py` (líneas 14–17), donde `POSTGRES_CONNECTION_STRING` se resuelve desde variable de entorno con un default apuntando al host `streamlit-postgres.postgres.database.azure.com`. El conector `get_engine()` en `services/db.py` (líneas 15–23) instancia un motor SQLAlchemy con `pool_pre_ping=True`, lo que sugiere que el diseño contemplaba conexiones de larga duración propias de un servicio desplegado, no de notebooks efímeros.

#### Componentes que dependen de PostgreSQL

| Consumidor | Archivo / función | Tabla | Propósito inferido |
|-----------|-------------------|-------|-------------------|
| KPIs sellout | `services/sellout_kpis.py` → `get_sellout_kpis()` | `sellout` | Métricas anuales de volumen y ventas |
| Partners de entrenamiento | `config.py` → `get_training_partners()` | `sellout` | Lista dinámica de trading partners (`TP`) |
| Categorías de SKU | `config.py` → `get_sku_categories()` | `iqsigma` | Enriquecer selectores en Prediction |
| Market Performance | `services/market_performance.py` | `iqsigma` | Ventas, unidades y share por marca |
| Precarga al inicio | `app.py` → `preload_section_data()` (líneas 57–92) | Ambas | Evitar latencia al cambiar pestañas |

Es relevante notar una **asimetría de diseño**: los gráficos de la pestaña Dashboard Performance en `components/dashboard.py` (`_render_sales_chart_only`, `_render_brand_bar_chart`, líneas 174–317) utilizan **series estáticas hardcodeadas**, mientras que los KPIs superiores sí dependen de la base. Esto indica una evolución por fases: primero UI con datos ilustrativos, después conexión parcial a datos reales.

#### Significado del host y del rechazo de credenciales

El FQDN `streamlit-postgres.postgres.database.azure.com` identifica una instancia de **Azure Database for PostgreSQL** cuyo nombre sugiere uso compartido con despliegues Streamlit del equipo original. En la auditoría de conectividad (Fase 1), el servidor **respondió en red** pero devolvió `password authentication failed`. Esta respuesta es diagnósticamente distinta de un timeout o un host inexistente: implica que (a) la instancia probablemente sigue activa o al menos el endpoint DNS resuelve, y (b) las credenciales embebidas en `config.py` (`postgres:postgres`) son **placeholders**, no credenciales operativas.

#### Diseño inferido del esquema relacional

A partir de las consultas SQL, puede inferirse que el modelo de datos fue pensado para analytics comercial en dos dominios complementarios:

**Dominio sellout (desempeño comercial por partner).** Las columnas `DATE`, `TP`, `QTY`, `Real_price` y `GROSS_SALES` en `services/sellout_kpis.py` orientan la tabla hacia ventas reales por trading partner, con volumen (`QTY`), precio unitario (`Real_price`) y venta bruta (`GROSS_SALES`). Las agregaciones anuales y comparaciones interanuales (CTEs en líneas 39–81) evidieren un panel de seguimiento de sellout para “Home Appliances - Training Partner Offer”, coherente con el título renderizado en `render_dashboard()` (`components/dashboard.py`, líneas 447–448).

**Dominio iqsigma (inteligencia de mercado).** Las columnas `DATE`, `BRAND`, `CATEGORY`, `SKU` y `PRICE_SOLD` en `services/market_performance.py` (SQL `BRAND_YEARLY_SQL` y `CATEGORY_BRAND_UNITS_SQL`) orientan la tabla hacia posicionamiento competitivo: ventas agregadas por marca, unidades por categoría-marca y derivación de precio promedio. La constante `WHIRLPOOL_FAMILY` (línea 11) agrupa marcas del ecosistema Whirlpool (`WHIRLPOOL`, `ACROS`, `MAYTAG`, `KITCHENAID`), lo que refuerza el propósito de **market share** y benchmarking.

#### Queries documentadas en código

**Tabla `sellout`:**

| Query | Archivo | Columnas usadas |
|-------|---------|----------------|
| Suma anual de `QTY` | `sellout_kpis.py:25–28` | `DATE`, `QTY` |
| Suma anual `QTY * Real_price` | `sellout_kpis.py:32–35` | `DATE`, `QTY`, `Real_price` |
| Delta interanual de `QTY` | `sellout_kpis.py:39–58` | `DATE`, `QTY` |
| Delta interanual de `GROSS_SALES` | `sellout_kpis.py:62–81` | `DATE`, `GROSS_SALES` |
| Partners distintos | `config.py:156–162` | `TP` |

**Tabla `iqsigma`:**

| Query | Archivo | Columnas usadas |
|-------|---------|----------------|
| Ventas/unidades anuales por marca | `market_performance.py:13–21` | `DATE`, `BRAND`, `PRICE_SOLD` |
| Unidades por categoría-marca | `market_performance.py:24–32` | `DATE`, `CATEGORY`, `BRAND` |
| Categoría por SKU | `config.py:92–96` | `SKU`, `CATEGORY` |

#### Comportamiento ante indisponibilidad

Cuando PostgreSQL falla, el sistema **degrada de forma silenciosa o parcial**:

- `get_sellout_kpis()` devuelve ceros y registra `error` en el diccionario (`sellout_kpis.py:137–149`).
- `get_training_partners()` cae en `DEFAULT_PARTNERS` (`config.py:172`).
- `get_sku_categories()` marca todos los SKUs como `"Unknown"` (`config.py:117–121`).
- `render_market_performance()` muestra error o mensaje vacío (`components/market_performance.py:199–205`).

La aplicación **no detiene el arranque**, pero pierde credibilidad analítica: KPIs en cero, Market Performance vacío y selectores de predicción empobrecidos.

#### Recuperación vs reemplazo

**Pedir al dueño original:** connection string válida con permisos de lectura, esquema confirmado, diccionario de columnas, export CSV/Parquet anonimizado de `sellout` e `iqsigma`, y ventana de disponibilidad del servidor.

**Alternativa sin acceso original:** implementar un `DataProvider` (Sección 10.6) que lea CSV/SQLite/PostgreSQL local con el mismo contrato de columnas. Los gráficos estáticos del dashboard pueden mantenerse temporalmente, pero los KPIs y Market Performance requieren datos sustitutos coherentes.

---

### 10.2 Datos `sellout` e `iqsigma`

#### Interpretación funcional de cada tabla

La tabla **`sellout`** parece modelar transacciones o registros de **sell-out por trading partner (TP)**: cuánto se vendió, a qué precio y en qué periodo. Su granularidad inferida es diaria o subanual (`DATE` casteado a `date` en SQL). Alimenta el discurso comercial interno: volumen, facturación y variación interanual por partner.

La tabla **`iqsigma`** —nombre posiblemente abreviado de “IQ Sigma” o dataset de inteligencia de precios— parece modelar **observaciones de mercado** a nivel SKU/marca/categoría, con `PRICE_SOLD` como proxy de valor de venta en canal. Alimenta el análisis competitivo: participación de Whirlpool frente a otras marcas, posición relativa y distribución por categoría.

#### Diferencia funcional

| Dimensión | `sellout` | `iqsigma` |
|---------|-----------|-----------|
| Pregunta de negocio | ¿Cómo vendemos por partner? | ¿Cómo nos posicionamos en el mercado? |
| Dashboard | Dashboard Performance (KPIs) | Market Performance |
| Granularidad clave | Partner (`TP`), tiempo | Marca, categoría, SKU |
| Métricas derivadas | Artículos vendidos, ventas, deltas | Market share, ventas Whirlpool, precio medio, ranking |
| Uso secundario | Partners en Prediction | Categorías en selectores SKU |

#### Métricas calculadas

**Desde `sellout`** (`get_sellout_kpis`):

- `articles_this_year`: Σ `QTY` del año corriente
- `sales_this_year`: Σ (`QTY` × `Real_price`)
- `articles_delta_percentage`: variación porcentual de `QTY` vs año anterior
- `sales_delta_percentage`: variación porcentual de `GROSS_SALES` vs año anterior

**Desde `iqsigma`** (`compute_latest_year_kpis`):

- `market_share`: ventas Whirlpool family / ventas totales mercado
- `whp_sales`, `whp_sales_delta`: ventas y variación relativa de la familia Whirlpool
- `avg_price`: ventas Whirlpool / unidades Whirlpool
- `position`: ranking competitivo por ventas
- Gráficos de tendencia y histograma por categoría (`components/market_performance.py`)

#### Contrato de datos propuesto

**Tabla `sellout` — columnas sugeridas:**

| Columna | Obligatoria | Uso en código | Simulable |
|---------|-----------|---------------|-----------|
| `DATE` | Sí | Filtros anuales, histórico XGBoost (`run_model.py:66`) | Sí (fechas sintéticas) |
| `TP` | Sí | KPIs, partners, predicción | Sí (lista `DEFAULT_PARTNERS`) |
| `SKU` | Sí | Predicción, validación | Sí (`unique_skus.txt`) |
| `QTY` | Sí | KPIs, features XGBoost | Sí |
| `Real_price` | Sí | KPIs, comparación predicción | Sí |
| `GROSS_SALES` | Sí | Delta ventas, feature XGBoost | Sí (derivable de QTY×precio) |
| `CATEGORY` | Condicional | Predicción XGBoost (`run_model.py:73–84`) | Sí (mapeo SKU→categoría) |
| `BRAND` | Opcional | No referenciada en SQL sellout | Sí |
| `INFLATION` | Opcional | Mostrada en statement (`run_model.py:74,108`) | Sí (default 0.0) |
| `INV` | Opcional | Feature XGBoost (`run_model.py:79`) | Sí (default 1) |

**Tabla `iqsigma` — columnas sugeridas:**

| Columna | Obligatoria | Uso en código | Simulable |
|---------|-----------|---------------|-----------|
| `DATE` | Sí | Agregaciones anuales | Sí |
| `BRAND` | Sí | KPIs mercado, gráficos | Sí |
| `CATEGORY` | Sí | Histograma, categorías SKU | Sí |
| `SKU` | Condicional | Lookup categorías | Sí (`unique_skus.txt`) |
| `PRICE_SOLD` | Sí | Ventas agregadas (= proxy valor) | Sí |
| `TP` / retailer | Opcional | No usada en SQL iqsigma actual | Sí |
| `REGION` | Opcional | No usada en SQL actual | Sí |

#### Datos locales vs faltantes

**Disponibles:** `unique_skus.txt` (468 SKUs), partners por defecto en `config.py`, gráficos estáticos, mocks de FastAPI.

**Faltantes:** filas históricas de ambas tablas, diccionario de negocio oficial, modelos entrenados.

**Utilidad de `unique_skus.txt`:** permite reconstruir **selectores, validaciones de entrada y prototipos de UI** en Prediction y Price Calculator, pero **no** histórico de precios ni categorías sin un dataset auxiliar.

#### CSV sustitutos temporales

- `data/sample_sellout.csv`: filas diarias/mensuales con `DATE, TP, SKU, QTY, Real_price, GROSS_SALES, CATEGORY, INFLATION`.
- `data/sample_iqsigma.csv`: filas con `DATE, BRAND, CATEGORY, SKU, PRICE_SOLD` distribuidas en marcas competidoras y familia Whirlpool.

---

### 10.3 Modelos `.pkl` de Azure Blob

#### Inventario de artefactos remotos

El proyecto contempla **dos familias de modelos pickle**, con propósitos distintos:

| Artefacto | Ruta blob inferida | Consumidor | Rol |
|----------|------------------|-------------|-----|
| `final_xgb_model.pkl` | `modelstoragest` / `models/` | `run_model.py:153` | Modelo XGBoost entrenado |
| `final_xgb_encoded_columns.pkl` | `modelstoragest` / `data/` | `run_model.py:154` | Columnas one-hot esperadas |
| `final_xgb_source_data.pkl` | `modelstoragest` / `data/` | `run_model.py:155` | Dataset histórico para lookup |
| `{partner}_model.pkl` | `modelstoragest` / `models/` | `azure_model_service.py:29–30` | Modelo LSTM legacy por TP |

Los tres primeros constituyen un **pipeline acoplado** para la pestaña Prediction. El cuarto alimenta el backend FastAPI (`backend.py:74–89`), hoy **legacy** respecto a la UI principal.

#### XGBoost vs LSTM legacy

**XGBoost (activo en UI).** `generate_price_prediction_statement()` (`run_model.py:136–181`) descarga modelo, columnas y dataframe fuente, localiza el último registro para (`TP`, `SKU`), construye un vector codificado con `pd.get_dummies` y genera la predicción de precio. El resultado se presenta como *price statement* en `components/dashboard.py` (`render_prediction_statement_card`, líneas 465–533).

**LSTM (legacy vía FastAPI).** `AzureModelService.load_model()` descarga `{partner}_model.pkl` y delega predicción a objeto serializado; si falla, `LSTMModel` en `ml/lstm_model.py` aplica fallback trend-based sin TensorFlow. Este flujo corresponde al Price Calculator (`components/price_calculator.py`), **no renderizado** en `app.py`.

#### Flujo de la pestaña Prediction

1. Usuario selecciona SKU, TP y fecha (`render_prediction_dashboard`, `dashboard.py:536–643`).
2. Se invoca `generate_price_prediction_statement(sku, tp, prediction_date)`.
3. Se descargan tres pickles remotos (cache LRU en `_load_remote_pickle`).
4. `price_comparison_table()` filtra histórico por TP+SKU, toma último precio real, codifica features categóricas, predice y calcula `% Change`.
5. UI muestra tarjeta de precio + tabla HTML estilizada (`render_prediction_statement_card`, `components/dashboard.py`, líneas 465–533).

Variables de negocio implícitas en el modelo: **TP, SKU, CATEGORY** como categóricas; **INV, QTY, GROSS_SALES** como numéricas; precio pasado vs predicho como salida interpretable para pricing comercial.

#### Interpretación de errores HTTP en Blob

| Código | Significado probable | Evidencia en auditoría |
|--------|-------------------|----------------------|
| **403 Forbidden** | SAS expirado, revocado o permisos insuficientes | XGB blobs en `run_model.py` (expiran ene. 2026) |
| **404 Not Found** | Blob inexistente o convención de nombre incorrecta | `{partner}_model.pkl` con SAS contenedor |

#### Recuperabilidad y riesgo de `pickle.load`

Sin acceso Azure, los `.pkl` **no son recuperables desde el repo** (`.gitignore` excluye `*.pkl` y `models/`). Opciones: solicitar SAS temporal al dueño, reentrenar localmente, o fallback.

**Riesgo de seguridad:** `pickle.load` sobre contenido remoto permite ejecución de objetos arbitrarios si el blob estuviera comprometido. Buenas prácticas: almacenar localmente, firmar artefactos, preferir formatos serializables (ONNX, JSON de metadatos) en futuras iteraciones.

#### Alternativas propuestas

1. **`models/` local** (ignorado por Git): copiar manualmente pickles obtenidos con autorización.
2. **`models/README.md`**: instrucciones de obtención, checksum, versión XGBoost.
3. **Variables de entorno** para rutas: `LOCAL_MODEL_DIR`, `LOCAL_XGB_MODEL`, etc.
4. **Reentrenamiento aproximado** con `sample_sellout.csv` si se conocen features.
5. **Fallback por reglas**: media móvil o último precio ± inflación cuando falte modelo.
6. **Modo demo**: banner visible “Datos/modelo sintéticos — no producción”.

---

### 10.4 SAS tokens, almacenamiento propio y estrategia segura

#### Qué es un SAS token en este proyecto

Un **Shared Access Signature (SAS)** es una credencial delegada que autoriza operaciones acotadas (aquí, lectura `sp=r`) sobre contenedores o blobs de Azure Storage durante un intervalo temporal (`st`, `se`). En `azure_model_service.py` (línea 30) se concatena como query string a la URL; en `run_model.py` (líneas 14–27) está **incrustado en constantes**.

#### Por qué no debe hardcodearse

Los tokens en `config.py` (líneas 28–30) y `run_model.py` (líneas 14–27) implican: exposición en Git, imposibilidad de rotación sin commit, expiración que **rompe descargas** (confirmado: HTTP 403), y ampliación de superficie de ataque si el repositorio es público.

#### Rotación y revocación

Rotar implica emitir nuevo SAS o regenerar Account Key subyacente; revocar invalida tokens expuestos. Cualquier token visible en historial Git debe considerarse **potencialmente comprometido**.

#### Comparación de mecanismos de configuración

| Mecanismo | Uso apropiado | En este repo |
|-----------|---------------|------------|
| SAS en código | No recomendado | Presente (riesgo alto) |
| Variables de entorno | Local / App Service | Parcial (`config.py`; no en `run_model.py`) |
| `.env` (no versionado) | Desarrollo local | Ausente |
| GitHub Secrets | CI/CD | Esperado por workflow |
| Connection string PG | Backend BD | Default embebido `<REDACTED>` |

#### Estrategia recomendada para tu fork

1. Crear **Storage Account propio** o usar `LOCAL_MODEL_DIR`.
2. Contenedor privado para modelos.
3. SAS de solo lectura con expiración acotada vía variable de entorno.
4. `.env` local; `.env.example` público sin secretos.
5. GitHub Secrets para despliegue.
6. Modo offline: `LOCAL_MODEL_DIR=models`.

#### Tabla comparativa de opciones

| Opción | Dificultad | Costo conceptual | Dependencia externa | Fidelidad | Seguridad | Recomendación |
|--------|-----------|------------------|---------------------|----------|-----------|---------------|
| **A: Recuperar Azure original** | Media | Acceso heredado / coordinación | Alta (dueño original) | Máxima | Baja si reutilizas secretos ajenos | Solo con autorización explícita |
| **B: Azure propio** | Alta | Suscripción Azure + Storage + PG | Media (tu tenant) | Alta (si migras datos/modelos) | Alta (control total) | Producción propia |
| **C: Modelos en disco local** | Baja–Media | Disco local | Baja | Media (mismos pickles si los obtienes) | Alta | **Desarrollo / demo offline** |
| **D: Modo demo sintético** | Baja | Tiempo de ingeniería | Nula | Baja (ilustrativa) | Alta | **Evaluación académica / prototipo** |

---

### 10.5 Secretos de despliegue para GitHub

#### Qué hace `.github/workflows/deploy.yml`

El workflow (disparador: push a `main` y `workflow_dispatch`) ejecuta: checkout → buildx → login Docker Hub → build/push imagen `linux/amd64` → login Azure (si hay credenciales) → `az webapp config container set` → restart Web App → imprimir URL.

Variables hardcodeadas en el workflow (líneas 8–11):

- `IMAGE_NAME: joecast208/whirpooldash`
- `WEBAPP_NAME: streamlit-app-demos`
- `RESOURCE_GROUP: st`

#### Secretos esperados (no presentes en repo)

| Secreto | Función |
|---------|---------|
| `AZURE_CREDENTIALS` | JSON Service Principal para `azure/login@v2` |
| `DOCKERHUB_USERNAME` | Usuario Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso Docker Hub |

`azure_creds.json` en raíz es **placeholder vacío** (0 bytes); el workflow espera el JSON en GitHub Secret, no en disco.

#### Por qué tu fork no despliega automáticamente

Sin secretos configurados en `narajoEmmanuel/whirpooldash`, el push a `main`:

- Podría **fallar** en login Docker Hub, o
- Construir imagen pero **no** actualizar Azure (pasos condicionados a `AZURE_CREDENTIALS != ''`).

Incluso con secretos, usar imagen/Web App del autor implica **despliegue sobre infraestructura ajena**.

#### Riesgos de reutilizar infraestructura del autor

- Sobrescribir imagen `joecast208/whirpooldash` sin permiso.
- Reiniciar Web App ajena `streamlit-app-demos`.
- Exponer secretos propios a un pipeline apuntando a recursos incorrectos.

#### Cambios necesarios para despliegue propio

| Elemento | Archivo | Cambio |
|----------|---------|--------|
| Imagen Docker | `deploy.yml:9` | Tu usuario/repo |
| Web App | `deploy.yml:10` | Tu App Service |
| Resource group | `deploy.yml:11` | Tu RG |
| Secretos | GitHub Settings | Tus SP, Docker tokens |
| App settings Azure | Portal / CLI | `POSTGRES_*`, `AZURE_BLOB_*`, `PORT` |
| Trigger | `deploy.yml:4–5` | Desactivar push hasta estar listo |

#### Significado de conceptos

- **`AZURE_CREDENTIALS`:** credencial JSON (`clientId`, `clientSecret`, `subscriptionId`, `tenantId`) de Service Principal.
- **`DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN`:** publicación de imagen contenedor.
- **`WEBAPP_NAME`:** nombre del Azure App Service destino.
- **`RESOURCE_GROUP`:** grupo de recursos Azure contenedor.
- **Imagen Docker:** artefacto desplegable; referenciado como `{IMAGE_NAME}:latest`.
- **App Service:** hospeda contenedor Streamlit puerto 8080 (`entrypoint.sh`).
- **Service Principal:** identidad no humana para CI/CD (`README_DEPLOY.md`, líneas 129–138).

#### Alternativas de despliegue

| Plataforma | Viabilidad | Consideraciones |
|-----------|------------|-----------------|
| **Streamlit Community Cloud** | Media | Secretos vía dashboard; verificar compatibilidad FastAPI embebido en hilo |
| **Render / Railwayway** | Media–Alta | Variables de entorno; posible servicio separado para FastAPI |
| **Azure App Service propio** | Alta | Alineado con Dockerfile existente |
| **Docker local** | Alta para demo | Sin CI; `docker run -p 8080:8080` |
| **GitHub Actions desactivado** | Recomendado interim | Hasta configurar secretos propios |

---

### 10.6 Solución alternativa aproximada: “Modo offline reproducible”

Se propone una arquitectura sustitutiva que preserve la **demostrabilidad funcional** del dashboard sin depender del Azure original. No pretende replicar cifras comerciales reales.

#### 1. Datos

- Crear `data/sample_sellout.csv` e `data/sample_iqsigma.csv` sintéticos pero estructuralmente válidos.
- Anclar SKUs a `unique_skus.txt`.
- Marcar claramente origen demo en UI.

#### 2. Base de datos

| Opción | Descripción | Adecuación |
|--------|-------------|------------|
| CSV + Pandas | `DataProvider.read_csv()` | Fase 1 — mínima fricción |
| SQLite | SQL local, reusar queries con adaptador | Fase 1–2 |
| PostgreSQL Docker | Paridad con producción | Fase 3 |

#### 3. Modelos

| Nivel | Enfoque |
|-------|---------|
| Simple | Fallback: último precio, media móvil o regla % inflación |
| Intermedio | XGBoost entrenado con `sample_sellout.csv` |
| Avanzado | Modelos propios en Blob/Azure propio |

#### 4. Configuración sugerida (`.env.example`)

```env
DATA_MODE=demo
POSTGRES_CONNECTION_STRING=
LOCAL_SELLOUT_PATH=data/sample_sellout.csv
LOCAL_IQSIGMA_PATH=data/sample_iqsigma.csv
LOCAL_MODEL_DIR=models
AZURE_BLOB_BASE_URL=
AZURE_BLOB_SAS_TOKEN=<REDACTED>
API_BASE_URL=http://localhost:8000
API_PORT=8000
```

#### 5. Refactor futuro recomendado (sin implementar en esta auditoría)

- Capa **`DataProvider`**: unifica PostgreSQL, CSV y SQLite.
- Capa **`ModelProvider`**: unifica Blob, disco local y fallback.
- Evitar SQL directo desde componentes Streamlit.
- Banner **`st.warning`** cuando `DATA_MODE=demo`.

#### 6. Resultado esperado

- Dashboard abre sin credenciales Azure.
- KPIs calculados desde CSV demo.
- Market Performance operativo con `sample_iqsigma.csv`.
- Prediction con modelo local o fallback.
- Proyecto evaluable académicamente con limitaciones explicitadas.

#### Diagrama — arquitectura alternativa

```mermaid
flowchart TB
    subgraph UI["Streamlit app.py"]
        Tabs[Dashboard / Market / Prediction]
    end

    subgraph Providers["Capa de abstracción propuesta"]
        DP[DataProvider]
        MP[ModelProvider]
    end

    subgraph DataSources["Fuentes de datos"]
        CSV[CSV locales]
        SQL[SQLite / PostgreSQL local]
        AZPG[(PostgreSQL Azure)]
    end

    subgraph ModelSources["Fuentes de modelos"]
        LocalPKL[models/*.pkl local]
        Blob[(Azure Blob propio)]
        Rules[Fallback por reglas]
    end

    subgraph Deploy["Despliegue"]
        LocalDocker[Docker local]
        OwnAzure[Tu Azure App Service]
        GH[GitHub Actions + tus Secrets]
    end

    Tabs --> DP
    Tabs --> MP
    DP --> CSV
    DP --> SQL
    DP --> AZPG
    MP --> LocalPKL
    MP --> Blob
    MP --> Rules
    UI --> Deploy
```

---

### 10.7 Plan técnico por fases

#### Fase 0: Seguridad y documentación

- No subir secretos; redactar con `<REDACTED>`.
- Crear `.env.example`.
- Añadir `azure_creds.json` a `.gitignore`.
- Documentar variables y modos (`demo`, `local`, `azure`).

#### Fase 1: Modo local mínimo

- CSV demo; `DataProvider` con Pandas.
- KPIs básicos y Market Performance local.
- Banner de modo demo.

#### Fase 2: Predicción aproximada

- Fallback por reglas.
- XGBoost local si hay datos suficientes.

#### Fase 3: Azure propio

- Storage + PostgreSQL propio (o Neon/Supabase).
- GitHub Secrets propios; workflow actualizado.

#### Fase 4: Limpieza profesional

- CI seguro; pruebas smoke; documentación final.
- Separación estricta demo/producción.

---

### 10.8 Preguntas concretas para el dueño original

1. ¿Pueden compartir export CSV anonimizado de `sellout` e `iqsigma`?
2. ¿Cuál es el diccionario de datos oficial de cada columna?
3. ¿Los blobs `final_xgb_*.pkl` y `{partner}_model.pkl` siguen en `modelstoragest`?
4. ¿Pueden emitir SAS temporal de solo lectura con expiración acotada?
5. ¿Qué versión de Python usaron en producción (Docker indica 3.11)?
6. ¿La Web App activa sigue siendo `streamlit-app-demos`?
7. ¿Qué App Settings tenía configurados (PG, SAS, `DATA_SOURCE_TYPE`)?
8. ¿El Price Calculator LSTM sigue activo o es legacy frente a XGBoost?
9. ¿Con qué features y target se entrenó XGBoost?
10. ¿La rama `main` refleja el estado final del proyecto o hay commits en ramas upstream no mergeadas?

#### Qué no debe pedirse o compartirse públicamente

- Passwords y connection strings completas.
- SAS tokens completos o Account Keys.
- Archivos `azure_creds.json` con secretos reales.
- GitHub Secrets del repo original.
- Dumps de datos sin anonimizar con precios/comerciales sensibles.

---

## 11. Estado actual de viabilidad

### 11.1 ¿Puedes trabajar hoy?

**Sí, con limitaciones claras.** El repositorio es viable como base técnica; la reproducibilidad plena requiere datos, modelos o sustitutos descritos en la Sección 10.

| Componente | Funciona sin credenciales | Notas |
|------------|----------------------|-------|
| Arranque Streamlit | Sí (tras `pip install`) | Backend en puerto 8000 |
| Sidebar / navegación / CSS | Sí | Logo local presente |
| Gráficos estáticos Dashboard | Sí | Datos hardcodeados |
| KPIs sellout | No (degradado a 0) | BD requerida para datos reales |
| Market Performance | No | BD requerida |
| Prediction XGBoost | No | Blobs 403 / SAS expirados |
| FastAPI + mock predict | Sí | vía `/docs` o cliente HTTP |
| Modo offline propuesto (Sección 10.6) | **Implementable** | Requiere desarrollo futuro |
| CI/CD en tu fork | No | Secretos e infra propios |

### 11.2 Qué funciona vs qué falla

**Funciona:** estructura app, imports, mock layer FastAPI, gráficos estáticos, lista SKUs, API local con fallback.  
**Falla o degrada:** PostgreSQL auth, blobs XGBoost, categorías SKU desde BD, métricas reales de mercado, despliegue automático sin configuración propia.

---

## 12. Riesgos técnicos

| Categoría | Riesgo | Severidad |
|-----------|-------|-----------|
| Seguridad | SAS tokens y connection strings en código fuente e historial Git | Alta |
| Seguridad | `azure_creds.json` versionado pero no en `.gitignore` | Media |
| Seguridad | `pickle.load` remoto sin validación de integridad | Media |
| Datos | Dependencia total de BD Azure no incluida en repo | Alta |
| Datos | Ausencia de capa `DataProvider` unificada | Media |
| ML | SAS expirados en `run_model.py` — predicción rota | Alta |
| ML | Métricas de evaluación inventadas en UI (`dashboard.py:654–663`) | Baja (misleading) |
| Dependencias | Sin lockfile; paquetes opcionales comentados | Media |
| Deuda técnica | Imports muertos (`price_calculator`, `sku_table` en `app.py`) | Baja |
| Deuda técnica | `DATA_SOURCE_TYPE=database` no implementado (`data_service.py:32–38`) | Media |
| Despliegue | Workflow apunta a recursos del autor | Alta |
| Despliegue | Push a `main` dispararía CI si hay secretos mal configurados | Media |
| Gobernanza | Reutilizar infraestructura ajena sin autorización | Alta |

---

## 13. Plan de siguientes pasos

Este plan operativo complementa el plan por fases de la Sección 10.7.

### 13.1 Pasos inmediatos

1. Crear `.venv` e instalar `requirements.txt`.
2. Crear `.env` local (no commitear) a partir de plantilla Sección 10.6.
3. Ejecutar `streamlit run app.py` y validar UI parcial.
4. Leer Sección 10 completa antes de solicitar accesos al dueño original.

### 13.2 Limpieza de seguridad

1. **No commitear** valores reales de SAS/password.
2. Revocar tokens expuestos en `config.py` y `run_model.py` (coordinar con administrador Azure).
3. Añadir a `.gitignore`: `azure_creds.json` (`.env` ya cubierto).
4. Evaluar limpieza de historial Git si tokens estuvieron vigentes.
5. Desactivar trigger push en `deploy.yml` hasta tener secretos propios.

### 13.3 Documentación

1. Actualizar `README.md` con modos demo/azure y estado XGBoost vs LSTM.
2. Publicar `.env.example` cuando se implemente modo offline.
3. Mantener esta auditoría como referencia interna.

### 13.4 Prueba local

1. `pip install -r requirements.txt`
2. `streamlit run app.py`
3. Probar `http://localhost:8000/docs`
4. Tras Fase 1 offline: validar KPIs con CSV demo.

### 13.5 Recuperación o reemplazo de Azure

- **Opción A:** acceso autorizado al Azure original (Sección 10.4, opción A).
- **Opción B:** modo offline reproducible (Sección 10.6) — **recomendada para empezar**.
- **Opción C:** Azure propio (Sección 10.4, opción B).

### 13.6 Preparación para tu GitHub (`origin`)

1. **No reutilizar** `joecast208` / `streamlit-app-demos` / secretos ajenos.
2. Crear secretos propios en GitHub.
3. Parametrizar `deploy.yml` con tus recursos.
4. Mantener `upstream` push disabled.

---

## 14. Anexos técnicos

### 14.1 Comandos ejecutados en auditoría

**Fase 1:**

```powershell
pwd
git rev-parse --show-toplevel
git status --short
git branch --show-current
git branch -a
git remote -v
git log --oneline --decorate --all -20
git config --get remote.upstream.pushurl
git rev-parse origin/main upstream/main HEAD
git log --all -S "sp=r&st=" --oneline -8
git log --all -S "streamlit-postgres" --oneline -5
git log --all --oneline -- azure_creds.json
git ls-files azure_creds.json unique_skus.txt
python --version
pip install python-dotenv sqlalchemy psycopg2-binary fastapi uvicorn xgboost
python -c "import backend; ..."
python -c "from services.db import run_query; ..."
python -c "import requests; ... HEAD blob URLs ..."
```

**Fase 2:** revisión estática ampliada de archivos listados en Sección 14.3; búsqueda repo-wide de términos (`sellout`, `iqsigma`, `modelstoragest`, `AZURE_*`, `DOCKERHUB`, etc.).

### 14.2 Hallazgos principales

**Fase 1:**

1. `origin` = tu fork; `upstream` push = DISABLED; `main` sincronizado en `f1658d9`.
2. Solo `origin/main`; upstream tiene 10 ramas adicionales.
3. App Streamlit + FastAPI embebido; predicción UI activa = XGBoost remoto.
4. KPIs y Market Performance requieren PostgreSQL Azure.
5. SAS XGB expirados (403); SAS contenedor en `config.py` aún responde pero blobs partner 404.
6. `azure_creds.json` vacío y versionado.
7. Secretos embebidos — riesgo alto.

**Fase 2 (integraciones críticas):**

8. PostgreSQL es fuente operativa; mock layer no alimenta UI principal.
9. `sellout` = comercial/partner; `iqsigma` = mercado/competencia.
10. Prediction requiere **tres** pickles coordinados + histórico con columnas extendidas (`INFLATION`, `INV`).
11. LSTM/Azure es legacy vía FastAPI; Price Calculator no visible en navegación.
12. Workflow CI/CD amarrado a infraestructura del autor — requiere reconfiguración total para tu fork.
13. Modo offline reproducible es la ruta recomendada antes de Azure propio.

### 14.3 Archivos revisados

**Fase 1:** árbol general del proyecto.

**Fase 2 (lectura profunda):**

```
config.py, services/db.py, services/sellout_kpis.py, services/market_performance.py
services/run_model.py, services/azure_model_service.py, services/api_client.py
backend.py, app.py
components/dashboard.py, components/market_performance.py, components/price_calculator.py
ml/lstm_model.py, ml/data_processor.py
data/mock_data_source.py, data/data_service.py (referenciado)
requirements.txt, .gitignore
.github/workflows/deploy.yml
README.md, README_DEPLOY.md, INSTALL.md
unique_skus.txt, azure_creds.json
```

### 14.4 Mapa de variables de entorno (referencia rápida)

```env
DATA_SOURCE_TYPE=mock
DATA_MODE=demo
POSTGRES_CONNECTION_STRING=postgresql://USER:PASSWORD@streamlit-postgres.postgres.database.azure.com/postgres?sslmode=require
LOCAL_SELLOUT_PATH=data/sample_sellout.csv
LOCAL_IQSIGMA_PATH=data/sample_iqsigma.csv
LOCAL_MODEL_DIR=models
API_BASE_URL=http://localhost:8000
API_PORT=8000
AZURE_BLOB_BASE_URL=https://modelstoragest.blob.core.windows.net/models
AZURE_BLOB_SAS_TOKEN=<REDACTED>
PORT=8080
```

### 14.5 Recursos Azure identificados (públicos en código/docs)

| Recurso | Valor |
|---------|-------|
| PostgreSQL host | `streamlit-postgres.postgres.database.azure.com` |
| Storage account | `modelstoragest` |
| Blob containers | `models`, `data` |
| Resource group | `st` |
| Web App | `streamlit-app-demos` |
| URL placeholder | `streamlit-app-demos-a0a8b9e7c7c5cfcu.canadacentral-01.azurewebsites.net` |
| Docker Hub image | `joecast208/whirpooldash` |

### 14.6 Referencias internas clave (estilo APA simplificado)

Las inferencias de esta auditoría se derivan de los artefactos siguientes: configuración de conexión (`config.py`); capa de acceso SQL (`services/db.py`); servicios de KPIs y mercado (`services/sellout_kpis.py`; `services/market_performance.py`); pipeline XGBoost (`services/run_model.py`); carga de modelos LSTM (`services/azure_model_service.py`); orquestación Streamlit (`app.py`); componentes de UI (`components/dashboard.py`; `components/market_performance.py`); workflow de despliegue (`.github/workflows/deploy.yml`); y guía de despliegue (`README_DEPLOY.md`).

---

*Documento ampliado en Fase 2 (2026-06-19). No se modificó código funcional, no se hizo commit, no se hizo push, y `upstream` no fue alterado.*
