# Revisión final de pendientes del repositorio whirpooldash

**Fecha de revisión:** 2026-06-20  
**Commit analizado:** `5324767` — *docs: documentar recursos externos y desactivar deploy automatico*  
**Alcance:** inspección estática, búsqueda de secretos, pruebas locales no destructivas y comparación con ramas `upstream`. **Sin cambios de código funcional.**

---

## 1. Resumen ejecutivo

El repositorio está **documentado y estabilizado a nivel de gobernanza** (README, auditoría, deploy manual), pero **no está listo para demostración offline reproducible**. La base de código en `main` es coherente con `upstream/main` más cinco commits de documentación propios.

**Listo para continuar:**

- Arquitectura entendida y documentada en `docs/auditoria-tecnica-whirpooldash.md` y `README.md`.
- Deploy automático desactivado; el workflow solo corre con `workflow_dispatch`.
- Dependencias core importables en el entorno probado.
- FastAPI backend importable con rutas esperadas.
- Errores de PostgreSQL y XGBoost son **previsibles** y no bloquean el arranque de imports.

**Falta antes del modo offline reproducible:**

1. **Seguridad:** tokens SAS y connection string embebidos en código versionado (`config.py`, `services/run_model.py`).
2. **Configuración:** no existe `.env.example`; variables `DATA_MODE`, `LOCAL_*` aún no implementadas en `config.py`.
3. **Datos:** no hay `sample_sellout.csv` ni `sample_iqsigma.csv`; no hay capa `DataProvider`.
4. **`.gitignore`:** `azure_creds.json` sigue versionado y no está ignorado.
5. **Degradación silenciosa:** KPIs devuelven ceros sin banner claro cuando PostgreSQL falla.

**Veredicto:** el siguiente paso debe ser **Fase 0 (seguridad + `.env.example` + `.gitignore`)** seguido de **Fase 1 (modo offline reproducible)**, no reconexión a Azure del autor.

---

## 2. Estado Git y remotos

| Elemento | Valor |
|----------|-------|
| Rama actual | `main` |
| Working tree | **Limpio** (sin cambios sin commitear al momento de la revisión) |
| HEAD | `5324767` |
| `origin` | `https://github.com/narajoEmmanuel/whirpooldash.git` (fetch/push) — **tu fork** |
| `upstream` | `https://github.com/joelvshimself/whirpooldash.git` (fetch); push = **`DISABLED`** |
| `upstream/main` | `f1658d9` — tu `main` está **5 commits por delante** (solo documentación y workflow) |

### Últimos commits en `main`

```
5324767 docs: documentar recursos externos y desactivar deploy automatico
61d2506 docs: actualizar README principal del proyecto
d64ba15 docs: ampliar analisis de integraciones criticas
cb64247 docs: ampliar analisis de integraciones criticas
65cba4f docs: agregar auditoria tecnica inicial del proyecto
```

README, auditoría y cambio de `deploy.yml` **ya están commiteados** en `origin/main`. No hay pendientes de commit detectados en esta revisión.

---

## 3. GitHub Actions

**Archivo:** `.github/workflows/deploy.yml`

| Aspecto | Estado |
|---------|--------|
| Trigger `push` | **Eliminado** — confirmado |
| Trigger activo | Solo `workflow_dispatch:` |
| `env` (IMAGE_NAME, WEBAPP_NAME, RESOURCE_GROUP) | **Sin cambios** — siguen apuntando al autor (`joecast208/whirpooldash`, `streamlit-app-demos`, `st`) |
| Jobs y steps | **Sin cambios** — build Docker Hub + deploy Azure condicionado a `AZURE_CREDENTIALS` |

### Riesgo residual

Si alguien ejecuta el workflow manualmente **con secretos configurados**, podría publicar en Docker Hub o tocar el App Service del autor. El fork no debería tener esos secretos configurados.

### ¿Renombrar o deshabilitar más fuerte el workflow?

| Opción | Pros | Contras |
|--------|------|---------|
| Dejar como está (`workflow_dispatch` only) | Referencia útil; fácil reactivar con recursos propios | Sigue visible en Actions; riesgo humano si se configuran secretos incorrectos |
| Renombrar a `deploy.yml.disabled` o mover a `docs/` | Reduce ejecución accidental | Pierde plantilla lista en la ruta estándar de GitHub |
| Añadir job guard (`if: false`) | Imposibilita ejecución sin borrar el archivo | Confuso; requiere editar YAML para reactivar |

**Recomendación:** mantener el estado actual hasta tener infraestructura propia; no renombrar en esta fase.

---

## 4. Seguridad y secretos

Búsqueda con `git grep` sobre archivos versionados. **No se reproducen valores completos.**

### Hallazgos críticos (código)

| Archivo | Línea aprox. | Tipo | Riesgo | Acción recomendada |
|---------|--------------|------|--------|-------------------|
| `config.py` | 14–16 | Connection string PostgreSQL (`postgresql://...@streamlit-postgres...`) con password placeholder | **Alto** — host expuesto; patrón de credencial en historial Git | Externalizar a `POSTGRES_CONNECTION_STRING`; eliminar default embebido en Fase 0 |
| `config.py` | 28–30 | Token SAS completo (`sp=r&...&sig=...`) para contenedor `models` | **Alto** — credencial de lectura en repo e historial | Revocar/rotar en Azure; mover a env var; limpiar historial si estuvo vigente |
| `services/run_model.py` | 14–27 | Tres URLs blob XGBoost con SAS por blob (`sig=...`) | **Alto** — tokens expirados pero expuestos; Prediction rota (403) | Externalizar URLs o usar `LOCAL_MODEL_DIR`; revocar tokens |

### Hallazgos de referencia (bajo riesgo)

| Archivo | Línea aprox. | Tipo | Riesgo | Acción |
|---------|--------------|------|--------|--------|
| `.github/workflows/deploy.yml` | 15, 29–30 | Referencias a `secrets.AZURE_CREDENTIALS`, `DOCKERHUB_*` | Bajo | No configurar secretos del autor en el fork |
| `README.md`, `README_DEPLOY.md`, auditoría | varias | Documentación de secretos con placeholders | Bajo | Mantener redacción sin valores reales |
| `docs/auditoria-tecnica-whirpooldash.md` | 1245 | Ejemplo `postgresql://USER:PASSWORD@...` | Bajo | Placeholder explícito; OK |

### Otros patrones buscados

| Patrón | Resultado |
|--------|-----------|
| `AccountKey` | No encontrado en archivos versionados |
| `DefaultEndpointsProtocol` | No encontrado |
| `clientSecret` | Solo en documentación (descripción de `AZURE_CREDENTIALS`) |
| `modelstoragest` | Host público en config y docs — esperado |

### Descarga de modelos

`models/lstm_model.pkl` respondió **200** con SAS de contenedor en prueba previa. **No se descargó ni se ejecutó `pickle.load()`** en esta revisión. Cualquier copia local debe hacerse con autorización explícita y almacenarse fuera de Git (`models/` ya está en `.gitignore`).

---

## 5. `.gitignore`

### Lo que ya cubre

| Patrón | Cubierto |
|--------|----------|
| `.env`, `.env.local` | Sí |
| `.venv`, `venv/`, `env/` | Sí |
| `__pycache__/`, `*.pyc`, `*.py[cod]` | Sí |
| `*.pkl`, `models/`, `*.h5`, `*.joblib` | Sí |
| `.streamlit/secrets.toml` | Sí |
| `.pytest_cache/`, logs | Sí |

### Lo que falta (documentar para Fase 0 — no modificado en esta revisión)

| Entrada sugerida | Motivo |
|------------------|--------|
| `azure_creds.json` | Archivo **versionado hoy** (0 bytes); riesgo si alguien lo llena y commitea |
| `*_temp.py`, `_connectivity_test_temp.py` | Scripts de prueba temporales |
| `.env.*` excepto `.env.example` | Variantes locales |
| `*.pem`, `*.key`, `credentials.json` | Prevención genérica |

---

## 6. Prueba local

**Entorno:** Python **3.13.5** global (sin `.venv` en el repo). Dockerfile usa **Python 3.11** — conviene alinear en Fase 1.

**No se ejecutó** `streamlit run app.py` de forma interactiva (proceso largo). Se probaron imports y servicios representativos.

### Comandos ejecutados

```powershell
python --version          # 3.13.5
python -m pip --version     # pip 25.3
python -m pip install -r requirements.txt
python -c "import streamlit, pandas, sqlalchemy, requests, fastapi, uvicorn, xgboost; print('imports ok')"
python -c "import backend; ..."   # rutas FastAPI
python -c "get_engine(); SELECT 1"  # PostgreSQL
python -c "requests.head(FINAL_MODEL_PATH)"  # blob XGB
python -c "get_sellout_kpis(); get_brand_yearly_stats()"  # servicios KPI
```

### Resultados

| Prueba | Resultado | ¿Esperado? |
|--------|-----------|------------|
| Imports core | **OK** | Sí |
| `backend` rutas | `/`, `/api/predict`, `/docs`, etc. | Sí |
| TensorFlow | Warning — fallback LSTM simple | Sí (TF opcional) |
| PostgreSQL `SELECT 1` | **FAILED** — autenticación rechazada | Sí (placeholder) |
| XGB blob HEAD | **403** | Sí (SAS expirado) |
| `get_sellout_kpis()` | Dict con **ceros** (degradación silenciosa) | Parcialmente — debería mostrar error en UI |
| `get_brand_yearly_stats()` | **OperationalError** | Sí |

### Comportamiento esperado al ejecutar Streamlit (inferido)

| Componente | Comportamiento probable |
|------------|-------------------------|
| UI Streamlit | Abre en `:8501` |
| FastAPI embebido | Arranca en hilo daemon `:8000` |
| KPIs | Ceros o warnings; no rompe arranque |
| Market Performance | Vacío o error |
| Prediction XGBoost | Error 403 al cargar pickles |
| Gráficos estáticos | Pueden renderizar |

**Recomendación:** crear `.venv` con Python 3.11 antes de desarrollo offline para paridad con Docker.

---

## 7. Recursos externos

Resultados alineados con `README.md` (secciones *Recursos externos identificados* y *Estado de pruebas externas*, prueba 2026-06-20). **No se repiten URLs con SAS.**

| Recurso | Estado documentado | Confirmación en revisión |
|---------|-------------------|-------------------------|
| Azure Web App pública | HTTP **200**, HTML Streamlit | Consistente — no implica datos/modelos OK |
| Google Colab | HTTP **200**, HTML | Consistente — contenido puede requerir login Google |
| PostgreSQL Azure | Auth **rechazada** con placeholder | Reconfirmado en prueba local |
| Blob `models/lstm_model.pkl` | **200** (~151 KB) | No re-descargado; requiere autorización para copia local |
| Blobs XGBoost (×3) | **403** | Reconfirmado (`xgb_model_head=403`) |
| Contenedor `data` | **403** con SAS de `models` | Documentado correctamente |

### Colab (`10qiK5uM7dD8yxUwEj2gzDSB24u5v7dcd`)

- Enlace documentado en README.
- URL responde HTTP 200.
- **No se inspeccionó** el contenido del notebook (requiere sesión Google).
- **Utilidad potencial:** entrenamiento o experimentación XGBoost/LSTM — revisar manualmente después con permiso.
- **No copiar** código ni secretos del Colab al repo en esta fase.

### Autorización para modelos Azure

Descargar `lstm_model.pkl` (accesible) o blobs XGBoost (403 con SAS actual) **requiere**:

1. Autorización del administrador Azure original, o
2. Storage/Azure propio, o
3. Reentrenamiento local con datos demo.

---

## 8. Ramas upstream

`git fetch upstream --prune` ejecutado. Ramas remotas en upstream: `main`, `3tabs`, `azuret`, `david/backend`, `english`, `filtros-skus`, `interface`, `sellinview`, `selloutview`, `skus-cat`, `ui`.

**Ninguna rama fue mergeada, checkout permanente ni modificada.**

Todas las ramas listadas están **detrás de tu `main`** en documentación y features actuales (tu `main` incluye auditoría, XGBoost en `run_model.py`, Market Performance, etc.). Los diffs `main..upstream/*` muestran **eliminación masiva** de archivos presentes en tu rama.

| Rama | Último commit (resumen) | ¿Útil revisar después? | Notas |
|------|-------------------------|------------------------|-------|
| `upstream/david/backend` | Merge + DB backend | **Sí — media prioridad** | Añade `data/database_data_source.py`, `data/models.py`, `DATABASE_SETUP.md`, `QUICK_START.md`. Posible referencia para `DataProvider`/modo database. Revisar sin merge; puede tener config distinta. |
| `upstream/ui` | Login + DB checks | **Sí — baja/media** | Login Streamlit, `DATABASE_SETUP.md`; **elimina** Dockerfile y workflow deploy. Interesante para auth futura, no para offline inmediato. |
| `upstream/azuret` | Azure deployed (antigua) | Baja | UI/sidebar antigua; pierde auditoría y market performance actual. |
| `upstream/sellinview` | Market + navigation temprana | Baja | Anterior a estado actual de `main`. |
| `upstream/selloutview` | Sellout skeleton | Baja | Muy anterior; sin KPIs actuales. |
| `upstream/3tabs` | 3 tabs + scripts correo | **Evitar por ahora** | Contiene scripts y listas de correos (`correos_listos_para_enviar.txt`, pasarelas). Riesgo de PII/spam; no mezclar. |
| `upstream/main` | Igual a base pre-docs | Referencia | Punto de partida del fork antes de tus commits. |

**Riesgo de secretos en ramas upstream:** cualquier rama antigua puede tener `config.py` o SAS distintos. Revisar con `git show upstream/<rama>:config.py` **sin pegar valores** antes de cherry-pick.

---

## 9. Dependencias

**Archivo:** `requirements.txt` (rangos mínimos, sin lockfile)

### Necesarios (usados activamente)

| Paquete | Uso |
|---------|-----|
| `streamlit` | UI |
| `pandas`, `numpy` | Datos |
| `sqlalchemy`, `psycopg2-binary` | PostgreSQL |
| `plotly` | Gráficos |
| `requests` | Blobs HTTP |
| `python-dotenv` | `.env` |
| `fastapi`, `uvicorn`, `starlette`, `pydantic` | Backend |
| `xgboost` | Prediction |

### Posiblemente no usados / peso extra

| Paquete | Observación |
|---------|-------------|
| `streamlit-custom-sidebar` | Declarado; **no importado** en código revisado |
| `streamlit-float` | Declarado; **no importado** |
| `annotated-doc`, `protobuf` | Dependencias transitivas / FastAPI; peso moderado |

### Opcionales comentados (no instalados)

`tensorflow`, `scikit-learn`, `altair`, `pyarrow`, `pydeck` — LSTM usa fallback sin TF.

### Compatibilidad Python

| Entorno | Versión | Resultado |
|---------|---------|-----------|
| Prueba local | 3.13.5 | Imports OK |
| Dockerfile | 3.11-slim | Referencia de producción |

**Riesgo:** divergencia 3.11 vs 3.13. Conviene `.venv` con 3.11 y, en fase futura, `requirements-lock.txt` o pins más estrictos.

---

## 10. README, auditoría y consistencia documental

Revisión cruzada de `README.md` y `docs/auditoria-tecnica-whirpooldash.md`.

| Afirmación | README | Auditoría | ¿Consistente? |
|------------|--------|-----------|---------------|
| App no funciona completa | Sí — estado parcial | Sí — tabla §1 | **Sí** |
| PostgreSQL no funciona con repo solo | Sí — auth requerida | Sí — FAIL auth | **Sí** |
| XGBoost no recuperado | Sí — 403, SAS expirado | Sí — §1, §10 | **Sí** |
| Modo offline recomendado | Sí — Fase 1–3 | Sí — §10.6–10.7 | **Sí** |
| Tokens completos en docs | No — placeholders/`sig=...` | No — REDACTADOS | **Sí** |
| Deploy automático desactivado | Implícito en despliegue | §13.2 recomienda desactivar push | **Sí** — README podría mencionar explícitamente `workflow_dispatch` only (mejora menor, no aplicada) |

**Contradicción menor:** auditoría §1 dice tokens XGBoost "expirados (403)"; README añade matiz de que `lstm_model.pkl` puede responder 200 — **complementario**, no contradictorio.

**No se modificaron** README ni auditoría en esta revisión.

---

## 11. Licencia y atribución

| Elemento | Estado |
|----------|--------|
| `LICENSE` | **MIT License**, Copyright (c) 2025 **Joel Vargas** |
| `README.md` | Enlaza a LICENSE |
| Fork | `origin` → `narajoEmmanuel/whirpooldash`; `upstream` → `joelvshimself/whirpooldash` |

### Recomendaciones (sin cambiar licencia)

1. **Mantener** el archivo `LICENSE` intacto en fork (requisito MIT).
2. **Considerar** una nota breve en README o en este documento: *"Fork mantenido por [tu usuario] para evaluación académica; código base de Joel Vargas et al."*
3. **No cambiar** a otra licencia sin permiso del titular de copyright original.

---

## 12. Pendientes priorizados

### Crítico

1. Eliminar o externalizar tokens SAS en `config.py` y `services/run_model.py`.
2. Eliminar connection string default embebida en `config.py`.
3. Crear `.env.example` y documentar variables sin valores reales.
4. Añadir `azure_creds.json` a `.gitignore` y dejar de versionar contenido sensible.

### Alto

5. Implementar modo offline: `DataProvider`, CSV demo, banner `DATA_MODE=demo`.
6. Crear `data/sample_sellout.csv` y `data/sample_iqsigma.csv`.
7. Degradación explícita en UI cuando PostgreSQL falla (no solo KPIs en cero).
8. `.venv` con Python 3.11 para alinear con Docker.

### Medio

9. Parametrizar `deploy.yml` con recursos propios cuando existan.
10. Revisar estáticamente `upstream/david/backend` para ideas de `database_data_source.py`.
11. Evaluar eliminación de deps no usadas (`streamlit-float`, `streamlit-custom-sidebar`).
12. Lockfile o pins de dependencias.

### Bajo

13. Revisar Colab manualmente para pipeline de entrenamiento.
14. Limpiar historial Git si tokens SAS estuvieron vigentes al commitearse.
15. Nota de fork/atribución en README.
16. Renombrar workflow solo si hay riesgo de ejecución accidental con secretos.

---

## 13. Próxima fase recomendada

Orden sugerido **antes de tocar lógica de negocio compleja**:

```
Fase 0 (inmediata)
├── .env.example
├── .gitignore (+ azure_creds.json)
├── Externalizar secretos hardcodeados (config.py, run_model.py)
└── python -m venv .venv con Python 3.11

Fase 1 (modo offline reproducible)
├── data/sample_sellout.csv + sample_iqsigma.csv
├── DataProvider (CSV → KPIs + Market Performance)
├── DATA_MODE / LOCAL_* en config.py
└── Banner st.warning en modo demo

Fase 2 (predicción aproximada)
├── ModelProvider / LOCAL_MODEL_DIR
├── Fallback por reglas
└── XGBoost local opcional con CSV demo

Fase 3 (opcional, después)
├── Azure / PostgreSQL propio
├── Blob storage propio + SAS rotados
└── deploy.yml + GitHub Secrets propios
```

**No recomendado como siguiente paso:** reconectar al PostgreSQL/Blob/Web App del autor, ejecutar deploy manual con infra ajena, o mergear ramas upstream sin revisión de secretos.

---

## Anexo: pruebas ejecutadas en esta revisión

| # | Prueba |
|---|--------|
| 1 | `git status`, `branch`, `remote -v`, `log -8` |
| 2 | Lectura estática `.github/workflows/deploy.yml` |
| 3 | `git grep` patrones de secretos |
| 4 | Revisión `.gitignore` y `git ls-files azure_creds.json` |
| 5 | `pip install -r requirements.txt` + imports |
| 6 | Import `backend`, PostgreSQL `SELECT 1`, blob HEAD XGB |
| 7 | `get_sellout_kpis()` / `get_brand_yearly_stats()` |
| 8 | `git fetch upstream`, `git branch -r`, diffs vs ramas upstream |
| 9 | Revisión `requirements.txt`, LICENSE, README vs auditoría |

**No ejecutado:** `streamlit run app.py` interactivo, `pickle.load()`, dumps PostgreSQL, descarga de modelos, inspección de contenido Colab.
