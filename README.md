# 📈 TrendSim-Leveraged-Portfolio

## Evaluación de Riesgo y Optimización de Cartera Apalancada mediante Simulación Monte Carlo y Backtest Histórico

Este proyecto es una herramienta de análisis cuantitativo diseñada para evaluar y optimizar carteras de inversión apalancadas (como las utilizadas en plataformas tipo Quantfury). El objetivo central es encontrar la asignación de activos que **maximice el Sharpe Ratio apalancado** mientras se implementa una **estrategia de gestión de riesgo activa** para mitigar la probabilidad de un *Margin Call* (liquidación).

El enfoque se basa en el **muestreo de tendencias históricas** para preservar la estructura de mercado (momentum y *volatility clustering*) y simular un comportamiento más realista que los modelos basados en caminatas aleatorias (Geometric Brownian Motion).

-----

## ⚙️ Estructura del Repositorio

El proyecto consta de dos *notebooks* principales que trabajan de forma complementaria:

| Archivo | Tipo de Análisis | Descripción |
| :--- | :--- | :--- |
| **`MonteCarloSimulator.ipynb`** | **Simulación y Optimización** | Flujo completo que realiza la optimización de pesos (Max Sharpe) y ejecuta miles de simulaciones Monte Carlo. Es el módulo principal para la **proyección de riesgo y rentabilidad a 5 años** con la estrategia activa de DCA condicional. |
| **`BacktestHistorical.ipynb`** | **Backtest Comparativo** | Módulo de validación que compara la **estrategia ACTIVA (CON DCA condicional)** frente a una **estrategia PASIVA (SIN DCA)** mediante el uso de múltiples ventanas deslizantes de 5 años. Confirma el valor de la gestión activa en periodos históricos específicos. |

-----

## 🎯 Metodología Clave: Innovación y Realismo

La precisión de las proyecciones se basa en dos componentes metodológicos cruciales:

### 1\. Simulación con Preservación de Momentum (Trend Sampling)

Para superar las limitaciones del muestreo diario aleatorio (que destruye la estructura temporal y subestima el riesgo de *crashes* prolongados), se utiliza un muestreo de tendencias:

  * **Extracción de Tendencias:** El código analiza los datos históricos (e.g., 7 años) y extrae ventanas de tiempo (tendencias) donde el movimiento total superó un umbral (e.g., $\pm 5\%$).
  * **Muestreo con `TrendSampler`:** En lugar de seleccionar un día al azar, la simulación selecciona una **tendencia completa** al azar y reproduce sus días consecutivos (*momentum*). Una vez que la tendencia termina, se selecciona otra.
  * **Corrección de Sesgo:** Se aplica un **Factor de Escalamiento Aritmético** a los retornos muestreados para asegurar que la volatilidad total simulada coincida con la volatilidad histórica de referencia, corrigiendo el sesgo de subrepresentación de días "neutrales" introducido por la extracción de solo tendencias extremas.

### 2\. Estrategia de DCA Condicional y Antirriesgo

La aportación mensual (`$2,000 USD`) no se despliega automáticamente, sino que funciona como un **buffer de capital** que se utiliza de forma condicional, priorizando la seguridad:

  * **Análisis de Margen Crítico:** Antes de desplegar capital, se evalúa el **Ratio de Margen Actual** ($\text{Equity} / \text{Exposure}$). Si el ratio cae por debajo de un umbral crítico ($\text{CRITICAL\_MARGIN}$), el $100\%$ del DCA se mantiene como *cash buffer* para proteger contra el *margin call*.
  * **Despliegue Gradual:** El capital solo se despliega (para aumentar el apalancamiento y rebalancear) si se cumplen condiciones de mercado favorables (e.g., *Drawdown* severo, alta **Desviación de Pesos** respecto al óptimo o **Volatilidad Realizada Baja**).
  * **Prioridad:** La estrategia prioriza la reducción del riesgo de liquidación sobre la maximización del crecimiento inmediato.

-----

## 📊 Métricas y Resultados Reportados

Ambos *notebooks* calculan y reportan las siguientes métricas clave para evaluar la estrategia:

| Métrica | Descripción | Enfoque de Riesgo |
| :--- | :--- | :--- |
| **Probabilidad de Margin Call** | Porcentaje de simulaciones (a 1 y 5 años) que resultan en la liquidación del capital. | Riesgo Extremo ($\text{Tail Risk}$). |
| **Capital Final y Percentiles (P10, P50, P90)** | La dispersión del capital final a 5 años, con un enfoque en el $\text{P10}$ (escenario adverso). | Rendimiento Proyectado. |
| **IRR (Internal Rate of Return)** | Rentabilidad anualizada, considerando los flujos de caja reales (inversión inicial + aportaciones mensuales). | Rentabilidad Comparable. |
| **Max Drawdown (Máxima Caída)** | Máxima caída porcentual del capital (Equity) desde su pico histórico en cada trayectoria. | Riesgo de Mercado. |
| **Sharpe Ratio** | Relación entre el retorno excedente (sobre la tasa libre de riesgo) y la volatilidad, calculado individualmente para cada trayectoria. | Eficiencia del Capital. |

-----

## 🛠️ Instalación y Ejecución

Para ejecutar este proyecto, necesitarás un entorno Python (preferiblemente un *notebook* de Colab o Jupyter) con las bibliotecas estándar de análisis de datos:

### 1\. Dependencias

Asegúrate de tener instaladas las siguientes librerías:

```bash
pip install -r requirements.txt
```

### 2\. Parámetros Iniciales

El comportamiento del modelo se controla mediante el diccionario `METAPARAMETERS` (definido en el Bloque 1 de cada *notebook*). Los parámetros críticos incluyen:

  * `"leverage"`: Factor de apalancamiento (e.g., $2.5 \text{x}$).
  * `"initial_capital"`: Capital de inicio.
  * `"monthly_contribution"`: Aportación mensual ($\text{DCA}$).
  * `"simulation_years"`: Horizonte de la simulación (e.g., $5$ años).
  * `"num_simulations"`: Número de trayectorias Monte Carlo (e.g., $10,000$).
  * `"maintenance_margin_ratio"`, `"safe_margin_ratio"`, `"critical_margin_ratio"`: Umbrales clave para la gestión de riesgo.

### 3\. Ejecución

Simplemente abre el *notebook* deseado (`MonteCarloSimulator.ipynb` o `BacktestHistorical.ipynb`) y ejecuta todas las celdas secuencialmente.

-----

## ✅ Requisitos del Entorno y Versiones

- **Python**: 3.11 (recomendado)
- **Jupyter**: Notebook/Lab
- Bibliotecas principales: `numpy`, `pandas`, `matplotlib`, `scipy`, `yfinance`
- Instala exactamente lo especificado en `requirements.txt` para evitar incompatibilidades.

Notas:
- `yfinance` desde 0.2.40 ha cambiado el valor por defecto de `auto_adjust=True`. El código ya contempla extracción robusta de `Adj Close`/`Close` para evitar problemas.
- Si usas entornos antiguos, actualiza `pip` y reinstala dependencias.

-----

## 🗃️ Fuentes de Datos

- Los precios se descargan de **Yahoo Finance** mediante `yfinance`.
- La extracción usa columnas `Adj Close` y, como respaldo, `Close` si fuese necesario.
- Puede haber huecos de datos o restricciones temporales; el código limpia columnas vacías y reporta tickers faltantes.
- Si aparece un error de descarga, reintenta más tarde o verifica conectividad/proxy.

-----

## 🚀 Guía Rápida: Backtest Histórico

1. Abre `BacktestHistorical.ipynb`.
2. Ejecuta la celda de configuración y descarga de datos (bloques iniciales).
3. Revisa el bloque de **Optimización** (pesos máximos por activo y Sharpe apalancado).
4. Ejecuta el bloque de **Backtests con ventanas deslizantes** (CON y SIN DCA).
5. Explora:
   - Tablas de rebalanceo mensual (P50)
   - Métricas comparativas (P50)
   - Gráficas de trayectorias y zonas de margen crítico
6. Opcional: usa la sección de **Single Simulation** para un inicio `año/mes` concreto.

-----

## 📚 Glosario Básico

- **DCA (Dollar-Cost Averaging)**: Aportaciones periódicas que se despliegan total o parcialmente según condiciones.
- **Leverage (Apalancamiento)**: Multiplicador de exposición sobre el capital propio.
- **Exposure (Exposición)**: Valor total de posiciones (apalancadas).
- **Equity (Capital)**: Valor neto del portafolio tras PnL.
- **Maintenance Margin Ratio**: Umbral mínimo de margen (equity/exposure) para evitar liquidación.
- **Margin Call**: Evento de liquidación cuando el margen cae por debajo del umbral de mantenimiento.
- **Drawdown**: Caída relativa desde el máximo histórico de equity.
- **Buffer de Margen**: Parte del DCA que se mantiene en efectivo para proteger el margen.

-----

## 🧪 Reproducibilidad

- Se fija `np.random.seed(42)` en la extracción de ventanas para resultados consistentes.
- Ventanas históricas no solapadas reducen sesgos por sobre-muestreo.
- El flujo separa: descarga/limpieza → optimización en train → evaluación en ventanas.

-----

## ⚠️ Limitaciones y Supuestos

- No se modelan explícitamente comisiones, deslizamientos, ni costes de financiación del apalancamiento.
- Puede existir **survivorship bias** y errores de datos de Yahoo Finance.
- Los parámetros (apalancamiento, umbrales de margen, factores de despliegue) impactan fuertemente el riesgo de liquidación.
- Los resultados históricos no garantizan rendimientos futuros.

-----

## 🧩 Troubleshooting

- `ValueError: No se pudieron descargar los datos...`: verifica conexión, rango de fechas y tickers.
- `Covarianza singular`: el código aplica una matriz diagonal de respaldo y continúa.
- `Faltan tickers`: se reportan como advertencia y se procede con los disponibles.
- Si Jupyter se congela, reinicia el kernel y vuelve a ejecutar secuencialmente.


-----

## *Este proyecto está diseñado únicamente con fines educativos y de investigación. No constituye asesoramiento financiero.*

-----

Apache License 2.0

(C) 2025 Charly López

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at:

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions
and limitations under the License.