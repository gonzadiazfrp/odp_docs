# Diagramas y Documentación del proyecto ODP

```mermaid
graph TD
    VISTA["🧾 especificacion_producto_politica_view.py"]
    SERVICIO["⚙️ EspecificacionProductoPoliticaService"]
    REPOSITORIO["📦 EspecificacionProductoRepository"]
    SESSION_REPO["💾 DjangoSessionRepository"]
    FUENTE_DATOS["🌐 specs_view (DB o microservicio)"]

    VISTA -->|usa| SERVICIO
    SERVICIO -->|usa| REPOSITORIO
    SERVICIO -->|usa| SESSION_REPO
    SESSION_REPO -->|lee/escribe| Datos_en_la_sesion
    REPOSITORIO -->|consulta| FUENTE_DATOS


```

| Nodo                                   | Rol                                                               |
|----------------------------------------|-------------------------------------------------------------------|
| `especificacion_producto_politica_view.py` | Vista o endpoint que maneja la petición del usuario               |
| `EspecificacionProductoPoliticaService`  | Lógica de negocio: decide si usar caché o ir al repo              |
| `EspecificacionProductoRepository`      | Encapsula acceso a datos (DB, query a `specs_view`, etc.)         |
| `DjangoSessionRepository`               | Guarda/recupera los datos en la sesión del usuario               |
| `specs_view`                           | Fuente de los datos reales (una vista en la DB)                  |


## Índice

1. [APIs del Modelo](#1-apis-del-modelo)  
2. [APIs de Datos](#2-apis-de-datos)  
3. [Visualización de Datos](#3-visualización-de-datos)
4. [Gestor de Políticas](https://github.com/gonzadiazfrp/odp_docs?tab=readme-ov-file#4-gestor-de-pol%C3%ADticas-policy-modal)

## 1. APIs del Modelo
## 🧠 Clase `OptimizationService` ->  [ODP_WEB_API]api/application/results/optimization/optimization_service.py

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
## 2. APIs de Datos
##  🗃️ app 'FastAPI' ->  [ODP]api/main_api.py

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

## 3. Visualización de Datos
## 🧠 [ODP_WEB_API] Trazabilidad y Edición de Gráficos de la App

Este documento describe el flujo de datos y la trazabilidad para los gráficos en el dashboard, así como las instrucciones para su modificación futura.

---

## 🔁 Flujo de datos

1. **Lógica de negocio**
   - 📄 Archivo: `api/domain/pages/dashboard/production_tons/production_tons_model.py`
   - Contiene el modelo `ProductionTonsModel` que define los campos de toneladas:
     - `total_toneladas_con_hueso`
     - `total_toneladas_sin_hueso`
     - `total_toneladas_hueso`
     - ...
   - Es el punto donde se estructuran los datos a visualizar.

2. **Servicio de implementación**
   - ⚙️ Archivo: `api/application/pages/dashboard/dashboard_page_service.py`
   - Método: `get_production_tons(request: Request) -> ProductionTonsModel | None`
   - Recupera los resultados optimizados (`model_result`) y los transforma en una instancia del modelo.
   - Esta función alimenta la vista con los datos finales que serán graficados.

3. **Renderizado HTML (Frontend)**
   - 📁 Archivo: `templates/pages/dashboard/index.html`
   - Fragmento relevante:
     ```html
     <div 
       hx-get="/api/dashboard/get_production_tons_graph/"
       hx-trigger="intersect once, modelExecuted from:body, sessionLoaded from:body"
       hx-swap="innerHTML"
       class="h-[566px] bg-white overflow-hidden rounded-md"
     >
       <div>Cargando...</div>
       <span class="loading loading-spinner text-primary"></span>
     </div>
     ```
   - Usa **HTMX** para hacer una solicitud al endpoint `/api/dashboard/get_production_tons_graph/`.
   - Este endpoint devuelve un fragmento HTML con una imagen renderizada del gráfico.

4. **Parcial HTML del gráfico**
   - 📄 Fragmento (ej. `partials/_production_tons_graph.html`)
   - Renderiza la imagen del gráfico como:
     ```html
     <img src="data:image/png;base64,{{ result.graph_base64 }}" alt="Gráfico de producción">
     ```

---

## 🎯 Cómo modificar el gráfico

### 1. Estilo visual desde el **HTML**
- Se puede modificar la presentación del gráfico (bordes, tamaño, espaciado, etc.) en `index.html` o el parcial.
- Ejemplo:
  ```html
  <img
    src="data:image/png;base64,{{ result.graph_base64 }}"
    class="w-full rounded-md shadow-lg border border-gray-200"
    alt="Gráfico de producción"
  >

### 2. Contenido del gráfico (colores, tipos, leyendas, ejes, etc.)

❗ **Se modifica desde el backend**, en el método que genera el gráfico  
(probablemente en `dashboard_page_service.py` o en algún módulo relacionado con gráficos).

Buscá una función que utilice **matplotlib**, **plotly** u otra librería para renderizar el gráfico y convertirlo en una imagen `base64`.

### Podés cambiar:
- Colores de barras
- Tipo de gráfico
- Títulos, leyendas, etiquetas
- Estilo de fuentes, tamaños, alineación

---

## 🧪 Verificación y Debugging

Para verificar que el gráfico se actualiza correctamente:

1. Ejecutar una sesión desde el dashboard.
2. Observar que se active el request:  
   `hx-get="/api/dashboard/get_production_tons_graph/"`
3. Confirmar que la imagen se actualiza con los nuevos datos.

Si la imagen **no** cambia:

- Verificar que el objeto `model_result` se esté actualizando correctamente.
- Asegurarse de que el gráfico se genere nuevamente en cada solicitud.

---
## 4. Gestor de Políticas (Policy Modal)

Este documento describe el funcionamiento completo del **Gestor de Políticas** en la aplicación, incluyendo el flujo de datos, la interacción entre el frontend y el backend, y los archivos involucrados.

---

## 📁 Archivos Involucrados

| Archivo                        | Rol Principal |
|-------------------------------|---------------|
| `router.py`                   | Lógica de backend, carga y actualización de datos |
| `_policy_modal.html`          | Contenedor del modal HTML |
| `_policy_modal_content.html`  | Contenido del modal con formularios y datos |
| `policy_modal.js`             | Lógica dinámica (eventos, validaciones, AJAX) |

---

## 🔄 Flujo de Funcionamiento

### 1. Apertura del Modal

- Desde el frontend, un botón (o acción) hace una solicitud a Django para abrir el modal de políticas.
- El contenido se carga dinámicamente usando **HTMX** y se inyecta en el `div` con ID `policy_modal_content`.


```html
<dialog id="policy_modal">
  <div id="policy_modal_content">
    <!-- Aquí se inyecta _policy_modal_content.html -->
  </div>
</dialog>
```

---

## 🔁 2. Backend: `router.py`

### 🔧 Función: `_prepare_policy_modal_data(request, pk)`
- Si `pk` es `None`, crea una política vacía.
- Si `pk` está definido, obtiene la política existente y sus relaciones:
  - Calidad de Hacienda
  - Cuarto
  - Entrada
  - Productos relacionados

También obtiene:
- Todos los cuartos (`all_cuartos`)
- Todas las entradas (`all_entradas`)
- Todas las calidades (`all_calidades`)
- Todos los destinos, productos y especificaciones

---

### 🌐 Función: `_render_policy_modal(data)`
- Usa el contexto preparado para renderizar el HTML parcial: `_policy_modal_content.html`

---

### ✅ Guardado: `_full_policy_update(payload, pk)`
- Valida datos de política y productos.
- Verifica que el rendimiento total coincida con el rendimiento de entrada.
- Actualiza o crea registros en la base de datos.

---

## 💻 3. Contenedor Modal: `_policy_modal.html`

- Contiene el `<dialog>` con `id="policy_modal"`.
- Escucha eventos `htmx:afterSwap` para abrir el modal cuando el contenido es cargado.
- Usa JS embebido para cerrar el modal, y lanzar el trigger personalizado `modalClosed`.

---

## 🧱 4. Contenido del Modal: `_policy_modal_content.html`

- Usa variables inyectadas desde Django (`{{ policy.nombre_politica }}`, etc.).
- Contiene:
  - Formulario de política
  - Selects dinámicos
  - Tabla con productos relacionados
  - Botones para guardar, filtrar, agregar productos, borrar selección

Incluye JS inline para:
- Validación de campos
- Activación del botón “Guardar” cuando hay cambios
- Cálculo de rendimiento total

---

## ⚙️ 5. Frontend JS: `policy_modal.js`

Controla toda la lógica interactiva del modal:

### ✅ Cargar Modal
```js
document.body.addEventListener('htmx:afterSwap', function (event) {
  if (event.detail.target.id === 'policy_modal_content') {
    initializeTable(); // carga la tabla
  }
});
```

---

### ✅ Guardar Cambios
```js
fetch(`/api/politicas/${policyId}/update_policy/`, {
  method: 'PUT',
  body: JSON.stringify({ policy, products })
});
```

- Recoge datos del formulario.
- Valida que existan productos válidos.
- Muestra errores con `showToast()` o `alert()`.
- Cierra el modal y refresca vista si todo sale bien.

---

### ❌ Eliminar Productos
```js
fetch(`/api/politicas/${policyId}/delete_products/`, {
  method: 'DELETE',
  body: formData
});
```

---

### 📊 Calcular Rendimiento Total
```js
function recalculateTotalRendimiento() {
  // Suma los valores de los inputs .rendimiento-field
}
```

---

### ➕ Agregar Fila de Producto
- Inserta una nueva fila con selects vacíos y campos listos para completar.

---

## 🧩 Datos Importantes

- Todos los `select`, `input`, y `checkboxes` están marcados con clases `.policy-field` o `.product-field`.
- El botón "Guardar" se activa solo cuando hay cambios detectados.
- HTMX facilita las actualizaciones parciales sin recargar toda la página.

---

## ✅ Validaciones Clave

### Backend
- Valida:
  - Campos obligatorios (nombre, cuarto, entrada, etc.)
  - Que el rendimiento total de productos coincida con el de entrada

### Frontend
- Bloquea acción si:
  - Faltan campos requeridos
  - No hay productos agregados
  - Hay errores de red o formato
