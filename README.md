# Diagramas y Documentación del proyecto ODP

# 🧠 Clase `OptimizationService` -> 🐍 [ODP_WEB_API]api/application/results/optimization/optimization_service.py

La clase `OptimizationService` es el núcleo del sistema de optimización. Se encarga de preparar, ejecutar y procesar los datos necesarios para optimizar recursos/productos en función de restricciones, costos y políticas. La clase OptimizationService es el centro del sistema.

Depende de dos componentes principales:

- OptmizationRepository, que ejecuta las consultas de negocio y el modelo.

- SessionRepository, que maneja los datos en sesión (caché, estado intermedio).

---

## 🏗️ Visión general

Esta clase:
- Toma datos de sesión y del repositorio.
- Arma estructuras de entrada para un modelo de optimización.
- Llama al modelo.
- Procesa y devuelve los resultados listos para ser usados.

---

## 🔒 Métodos privados

### `__get_parameters(request)`
Arma los parámetros generales del modelo:

- `cantidad_dias_habiles_mes`
- `capacidad_despostada_diaria`
- `dolar_blend`
- `mano_de_obra`
- `produccion_maxima_mensual`

Fuente: `SessionRepository`.

---

### `__update_entities_with_inputs(entities, datos_hacienda)`
Actualiza los datos de entrada (`entities`) con información real de hacienda:

- Peso, cantidad mínima/máxima, precio, porcentajes por calidad.
- Enlaza categorías y calidades según su `fk_categoria_hacienda`.

---

### `__get_change(item, changes)`
Devuelve un cambio registrado en caché si el índice coincide con el `item` actual.

---

## 🧩 Métodos públicos

### `apply_changes(request, entities, dolar_blend)`
🔧 Método principal de preparación.

**Pasos:**
1. Actualiza entidades con datos de hacienda.
2. Trae especificaciones de productos.
3. Crea el request para el maestro de producto/política.
4. Lo consulta (con caché y cambios aplicados).
5. Calcula las cuotas por producto y política.
6. Integra las políticas con las cuotas y categorías.
7. Devuelve:
   - Parámetros de entrada (`OptimizationParametersRequestModel`)
   - Entidades actualizadas
   - Integración de políticas
   - Cuotas por producto/política

---

### `run_model(request_data)`
Ejecuta el modelo optimizador de forma **asíncrona**:

- Devuelve `OptimizationResponseModel` o `OptmizationErrorModel`.

---

### `get_maestro_especificacion_producto_politica(request, request_data)`
Obtiene el **maestro de políticas por producto**:

- Lo busca primero en sesión (caché).
- Si no está, lo consulta.
- Aplica cambios personalizados y normaliza porcentajes.

---

### `get_ds_producto_politica_cuota(request, entities, maestro)`
Calcula las **cuotas por producto/política** con:

- Parámetros de entrada
- Entidades actualizadas
- Maestro

Resultado: `DSProductoPoliticaCuotaResponseModel`

---

### `get_ds_integracion_politica(request, cuotas, categorias, politicas, entradas_res)`
Integra todo en políticas finales:

- Usa un request armado con cuotas, categorías, políticas y entradas.
- Aplica porcentajes desde caché si existen.

Resultado: `List[IntegracionPoliticaModel]`

---

### `get_detalle_integracion_politica(request, cuotas, integracion, dolar_blend, fk_politica)`
Devuelve el detalle de una política específica.

- Si ya está cacheado, lo devuelve.
- Si no, lo solicita al repositorio.
- Se guarda para futuras consultas.

Resultado: `DetalleIntegracionPoliticaModel`

---

## 📦 Modelos involucrados

- `OptimizationRequestModel`
- `OptimizationResponseModel`
- `OptimizationEntitiesRequestModel`
- `OptimizationParametersRequestModel`
- `MaestroEspecificacionProductoPoliticaRequest`
- `MaestroEspecificacionProductoPoliticaItem`
- `DSProductoPoliticaCuotaRequestModel`
- `DSProductoPoliticaCuotaResponseModel`
- `DSProductoPoliticaCuotaModel`
- `DSIntegracionPoliticaRequestModel`
- `IntegracionPoliticaModel`
- `DetalleIntegracionPoliticaRequestModel`
- `DetalleIntegracionPoliticaResponseModel`
- `DetalleIntegracionPoliticaModel`

---

## 🧠 Resumen general

| Método                           | Función clave                                                  |
|----------------------------------|----------------------------------------------------------------|
| `__get_parameters`               | Arma parámetros globales del modelo                            |
| `__update_entities_with_inputs`  | Completa entidades con datos reales de hacienda                |
| `apply_changes`                  | Prepara toda la estructura previa a la optimización            |
| `run_model`                      | Ejecuta el modelo optimizador                                  |
| `get_maestro_especificacion_producto_politica` | Consulta/actualiza el maestro producto-política         |
| `get_ds_producto_politica_cuota` | Calcula cuotas por política y producto                         |
| `get_ds_integracion_politica`    | Integra datos con políticas definidas                          |
| `get_detalle_integracion_politica` | Muestra detalle por política específica                       |

---

💬 *Este servicio orquesta todo el flujo de la lógica de optimización, asegurándose de que los datos estén actualizados, consistentes y correctamente integrados antes y después de ejecutar el modelo.*





```mermaid
classDiagram
    class OptimizationService {
        -repository: OptmizationRepository
        -session_repository: SessionRepository
        +apply_changes(Request, OptimizationEntitiesRequestModel, float): Tuple
        +run_model(OptimizationRequestModel): OptimizationResponseModel | OptmizationErrorModel
        +get_maestro_especificacion_producto_politica(Request, MaestroEspecificacionProductoPoliticaRequest): List
        +get_ds_producto_politica_cuota(Request, OptimizationEntitiesRequestModel, List): DSProductoPoliticaCuotaResponseModel
        +get_ds_integracion_politica(Request, List, List, List, List): List
        +get_detalle_integracion_politica(Request, List, List, float, int): DetalleIntegracionPoliticaModel
    }

    class OptmizationRepository {
        +run_model(OptimizationRequestModel): OptimizationResponseModel | OptmizationErrorModel
        +get_maestro_especificacion_producto_politica(MaestroEspecificacionProductoPoliticaRequest): List
        +get_ds_producto_politica_cuota(DSProductoPoliticaCuotaRequestModel): DSProductoPoliticaCuotaResponseModel
        +get_ds_integracion_politica(DSIntegracionPoliticaRequestModel): List
        +get_detalle_integracion_politica(DetalleIntegracionPoliticaRequestModel): DetalleIntegracionPoliticaResponseModel
    }

    class SessionRepository {
        +get_capacidades(Request): RestrictionsCapacityModel
        +get_dolar(Request): DolarModel
        +get_mano_de_obra(Request): LaborCostsModel
        +get_datos_hacienda(Request): List
        +get_especificacion_productos(Request): List
        +get_maestro_producto_politica(Request): List
        +set_maestro_producto_politica(Request, List)
        +get_maestro_producto_politica_changes(Request): List
        +get_integrations_percentages(Request): List
        +set_ds_producto_politica_cuota(Request, List)
        +set_maestro_producto_politica_cuota(Request, List)
        +set_integrations_by_policy(Request, List)
        +get_detalle_integracion_politica(Request, int): DetalleIntegracionPoliticaModel
        +set_detalle_integracion_politica(Request, DetalleIntegracionPoliticaModel)
    }



    OptimizationService --> OptmizationRepository : uses
    OptimizationService --> SessionRepository : uses

        SessionRepository --> RestrictionsCapacityModel
    SessionRepository --> DolarModel
    SessionRepository --> LaborCostsModel
    SessionRepository --> FarmDataModel
    SessionRepository --> FarmCategoryModel
    SessionRepository --> BeefIntakeModel
    SessionRepository --> PolicyModel

```

---

#  🗃️ app 'FastAPI' -> 🐍 [ODP]api/main_api.py

## 🧠 Descripción General

Esta API permite generar los datasets necesarios y ejecutar un modelo de optimización de políticas para producción ganadera. A través de distintos endpoints se procesan datos, se realizan validaciones, integraciones y transformaciones hasta obtener los resultados del modelo optimizado.

---
## 📌 Endpoints Clave

### 🔹 `/get_maestro_especificacion_producto_politica`

- **Método:** `POST`
- **Input:** 
  - `especificacion_producto`
  - `dolar_blend`
  - `entities`
- **Output:** 
  - `maestro_especificacion_producto_politica`
- **Uso:** Paso previo para generar la relación producto-política-cuota.

---

### 🔹 `/get_ds_producto_politica_cuota`

- **Método:** `POST`
- **Input:**
  - `maestro_especificacion_producto_politica`
  - `entities`
  - `input_values:` Valores de entrada básicos para el modelo de optimización. Incluye parámetros operativos y financieros necesarios para los cálculos
- **Output:**
  - `ds_producto_politica_cuota`
  - `entidad_politica`
  - `input_values`
- **Uso:** Relaciona productos con políticas y calcula capacidades productivas. Crea el maestro de producto política calculando todas las métricas asociadas como el total de costos, el total de impuestos, precio neto, integración por producto, etc.

---

### 🔹 `/get_ds_integracion_politica`

- **Método:** `POST`
- **Input:**
  - `ds_producto_politica_cuota:` Dataset que establece qué productos están asociados a qué políticas y cuotas específicas.
  - `entidad_categoria_hacienda:` Lista de categorías de hacienda con sus atributos. Contiene información sobre las diferentes categorías de ganado disponibles.
  - `entidad_politica:` Lista de políticas de producción con sus características. Define las diferentes políticas de producción disponibles.
  - `entidad_entrada_res:` Lista de entradas de res con sus características. Define las diferentes entradas de res disponibles.
- **Output:** 
  - `ds_integracion_politica`
- **Uso:** Integra políticas considerando entradas, cuotas y restricciones.


