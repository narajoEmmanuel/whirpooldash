# Auditoría técnica del proyecto whirpooldash

**Fecha de auditoría:** 2026-06-19  
**Ruta del repositorio:** `C:\Users\naran\OneDrive - Instituto Tecnologico y de Estudios Superiores de Monterrey\IMD 7\whirpooldash`  
**Commit analizado:** `f1658d9` — *feat: update dashboard styling and add new partner*

---

## 1. Resumen ejecutivo

**whirpooldash** es un dashboard interno de Whirlpool construido con **Streamlit** como interfaz principal y un **backend FastAPI** embebido para predicción de precios. El proyecto combina visualización de KPIs de sellout, análisis de desempeño de mercado (tabla `iqsigma`) y una pestaña de **predicción XGBoost** que descarga modelos y datos desde **Azure Blob Storage**. También existe integración con **Azure Database for PostgreSQL**, aunque los datos operativos no están en el repositorio.

### ¿Está funcional hoy?

| Área | Estado |
|------|--------|
| UI Streamlit (arranque general) | Parcial — arranca tras instalar dependencias |
| Dashboard Performance (KPIs) | Depende de PostgreSQL Azure — **falla sin credenciales válidas** |
| Dashboard Performance (gráficos) | **Funciona** — datos estáticos hardcodeados en código |
| Market Performance | Depende de PostgreSQL — **sin datos locales alternativos** |
| Prediction (XGBoost) | Depende de blobs Azure — **tokens SAS expirados (HTTP 403)** |
| Price Calculator (FastAPI + Azure LSTM) | Backend importable; UI **no expuesta** en navegación actual |
| CI/CD y despliegue | Workflow presente; requiere secretos externos no configurados en tu fork |

### Conclusión de viabilidad

Puedes **continuar trabajando** en el proyecto para entender la arquitectura, mejorar UI, documentación y preparar tu propio despliegue. Para funcionalidad completa necesitas:

1. Credenciales válidas de **PostgreSQL Azure** (`streamlit-postgres.postgres.database.azure.com`).
2. **Tokens SAS renovados** o modelos/datos descargados localmente para Azure Blob (`modelstoragest`).
3. Secretos de **GitHub Actions** (`AZURE_CREDENTIALS`, `DOCKERHUB_*`) si quieres desplegar.

Sin credenciales, puedes ejecutar Streamlit y ver gráficos estáticos del dashboard, pero KPIs reales, Market Performance y predicciones XGBoost no funcionarán.

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

---

## 10. Estado actual de viabilidad

### 10.1 ¿Puedes trabajar hoy?

**Sí, con limitaciones claras.**

| Componente | Funciona sin credenciales | Notas |
|------------|----------------------|-------|
| Arranque Streamlit | Sí (tras `pip install`) | Backend en puerto 8000 |
| Sidebar / navegación / CSS | Sí | Logo local presente |
| Gráficos estáticos Dashboard | Sí | Datos hardcodeados |
| KPIs sellout | No | BD requerida |
| Market Performance | No | BD requerida |
| Prediction XGBoost | No | Blobs 403 |
| FastAPI + mock predict | Sí | vía `/docs` o cliente HTTP |
| Docker local | No verificado | Docker no instalado en máquina auditada |
| CI/CD en tu fork | No | Secretos no configurados |

### 10.2 Qué funciona vs qué falla

**Funciona:** estructura app, imports, mock layer, gráficos estáticos, lista SKUs, API local con fallback.  
**Falla o degrada:** PostgreSQL auth, descarga blobs XGBoost, categorías SKU desde BD, métricas reales de mercado.

---

## 11. Riesgos técnicos

| Categoría | Riesgo | Severidad |
|-----------|-------|-----------|
| Seguridad | SAS tokens y connection strings en código fuente e historial Git | Alta |
| Seguridad | `azure_creds.json` versionado pero no en `.gitignore` | Media |
| Datos | Dependencia total de BD Azure no incluida en repo | Alta |
| ML | SAS expirados en `run_model.py` — predicción rota | Alta |
| ML | `pickle.load` remoto — riesgo si blobs no confiables | Media |
| Dependencias | Sin lockfile; paquetes opcionales comentados | Media |
| Deuda técnica | Imports muertos (`price_calculator`, `sku_table` en `app.py`) | Baja |
| Deuda técnica | `DATA_SOURCE_TYPE=database` no implementado | Media |
| Deuda técnica | Métricas de evaluación inventadas en UI | Baja (misleading) |
| Despliegue | Workflow apunta a recursos del autor (`joecast208`, `streamlit-app-demos`) | Alta |
| Despliegue | Push a `main` dispararía CI si hay secretos | Media |

---

## 12. Plan de siguientes pasos

### 12.1 Pasos inmediatos

1. Crear `.venv` e instalar `requirements.txt`.
2. Crear `.env` local (no commitear) con variables vacías/plantilla.
3. Ejecutar `streamlit run app.py` y validar UI parcial.
4. Revisar este documento y decidir alcance (offline vs Azure real).

### 12.2 Limpieza de seguridad

1. **No commitear** valores reales de SAS/password.
2. Revocar SAS tokens expuestos en `config.py` y `run_model.py` (coordinar con dueño Azure).
3. Añadir a `.gitignore`: `azure_creds.json`, `.env` (ya parcialmente cubierto).
4. Considerar `git filter-repo` / BFG para limpiar historial si tokens estuvieron vigentes (requiere coordinación).
5. Rotar password PostgreSQL en Azure si password real alguna vez estuvo en repo.

### 12.3 Documentación

1. Actualizar `README.md` con estado real (XGBoost activo, LSTM legacy, deps de BD).
2. Documentar variables de entorno obligatorias.
3. Añadir sección "modo offline" con mocks.

### 12.4 Prueba local

1. `pip install -r requirements.txt`
2. `streamlit run app.py`
3. Probar `http://localhost:8000/docs` → POST `/api/predict` con body JSON
4. Opcional: export CSV de muestra para simular BD

### 12.5 Recuperación o reemplazo de Azure

**Opción A — Acceso real:** solicitar credenciales al autor; configurar `.env`.  
**Opción B — Offline:** export PostgreSQL + descarga manual de `.pkl` con SAS renovado; servir localmente.  
**Opción C — Reemplazo:** nuevo storage account + reentrenar XGBoost; nueva instancia PostgreSQL.

### 12.6 Preparación para tu GitHub (`origin`)

1. **No reutilizar** secretos/recursos de `joecast208` / `streamlit-app-demos`.
2. Crear tus secretos: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `AZURE_CREDENTIALS`.
3. Editar `.github/workflows/deploy.yml` con tu imagen, Web App y resource group.
4. Desactivar workflow hasta tener secretos (`workflow_dispatch` only o borrar trigger push).
5. Mantener `upstream` push disabled; usar fetch + merge manual cuando necesites.

---

## 13. Anexos técnicos

### 13.1 Comandos ejecutados en auditoría

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

### 13.2 Hallazgos principales

1. `origin` = tu fork; `upstream` push = DISABLED; `main` sincronizado en `f1658d9`.
2. Solo `origin/main`; upstream tiene 10 ramas adicionales de referencia.
3. App Streamlit + FastAPI embebido; predicción activa = XGBoost remoto.
4. KPIs y Market Performance requieren PostgreSQL Azure (auth fallida con defaults).
5. SAS tokens XGB en `run_model.py` expirados (HTTP 403); token en `config.py` aún responde pero partner model 404.
6. `azure_creds.json` vacío y versionado; `.gitignore` no lo excluye.
7. Secretos embebidos en código — riesgo de seguridad alto.
8. Docker no disponible en máquina auditada; Streamlit local viable tras instalar deps.

### 13.3 Archivos revisados

```
app.py, backend.py, config.py, requirements.txt
Dockerfile, entrypoint.sh, .gitignore, .dockerignore
README.md, README_DEPLOY.md, INSTALL.md
unique_skus.txt, azure_creds.json
.github/workflows/deploy.yml
components/dashboard.py, market_performance.py, price_calculator.py, sku_table.py
services/db.py, data_service.py, sellout_kpis.py, market_performance.py
services/azure_model_service.py, run_model.py, api_client.py
data/data_source.py, mock_data_source.py
ml/lstm_model.py, data_processor.py
utils/helpers.py
assets/whirpool_logo.png
```

### 13.4 Mapa de variables de entorno (referencia rápida)

```env
DATA_SOURCE_TYPE=mock
POSTGRES_CONNECTION_STRING=postgresql://USER:PASSWORD@streamlit-postgres.postgres.database.azure.com/postgres?sslmode=require
API_BASE_URL=http://localhost:8000
API_PORT=8000
AZURE_BLOB_BASE_URL=https://modelstoragest.blob.core.windows.net/models
AZURE_BLOB_SAS_TOKEN=<REDACTED>
PORT=8080
```

### 13.5 Recursos Azure identificados (públicos en código/docs)

| Recurso | Valor |
|---------|-------|
| PostgreSQL host | `streamlit-postgres.postgres.database.azure.com` |
| Storage account | `modelstoragest` |
| Blob containers | `models`, `data` |
| Resource group | `st` |
| Web App | `streamlit-app-demos` |
| URL placeholder | `streamlit-app-demos-a0a8b9e7c7c5cfcu.canadacentral-01.azurewebsites.net` |
| Docker Hub image | `joecast208/whirpooldash` |

---

*Documento generado como auditoría de lectura. No se modificó código funcional, no se hizo commit, no se hizo push, y `upstream` no fue alterado.*
