REGLAS PARA LA CONSTRUCCIÓN DEL DATASET SINTÉTICO - CAPBOX - Arturo - Jonathan - Karen
================================================================

1. CONFIGURACIÓN GENERAL
========================

1.1 Semillas de Reproducibilidad:
- Python random.seed(42)
- NumPy np.random.seed(42)
- Faker('es_ES') con configuración española

1.2 Objetivos de Tamaño:
- TARGET_BOXERS: 100,000 boxeadores
- TARGET_COMBATS: 50,000 combates históricos
- Dataset base inicial: 5,000 boxeadores

2. REGLAS DE GENERACIÓN DE BOXEADORES SINTÉTICOS
===============================================

2.1 Identificación y Datos Básicos:
------------------------------------
- id_atleta: UUID único generado con uuid.uuid4()
- nombre: Generado con Faker('es_ES').name()
- edad: Distribución normal (μ=26, σ=6), limitada entre [18, 45] años
- peso: Distribución uniforme entre [47, 125] kg
- estatura: Distribución uniforme entre [1.55, 2.10] metros

2.2 Características Demográficas:
---------------------------------
- guardia: ['diestra', 'zurda'] con probabilidades [0.85, 0.15] (85% diestros)
- nivel: ['amateur', 'pro'] con probabilidades [0.3, 0.7] (70% profesionales)
- division: Selección aleatoria entre 8 divisiones de peso estándar
  ['peso_mosca', 'peso_gallo', 'peso_pluma', 'peso_ligero', 
   'peso_welter', 'peso_medio', 'peso_mediopesado', 'peso_pesado']
- pais: Distribución entre 7 países principales:
  ['USA', 'España', 'México', 'UK', 'Brasil', 'Argentina', 'Colombia']
- anos_experiencia: Distribución exponencial (λ=8), mínimo 1 año

2.3 Métricas de Rendimiento (Valores Crudos):
----------------------------------------------
- precision_jab_raw: Normal (μ=15, σ=5), limitado entre [1, 30]
- combinacion_123_raw: Normal (μ=20, σ=6), limitado entre [1, 30]
- bloqueo_reflejo_raw: Normal (μ=10, σ=4), limitado entre [1, 20]
- resistencia_golpes_raw: Normal (μ=90, σ=25), limitado entre [30, 150]
- golpes_por_minuto_raw: Normal (μ=120, σ=30), limitado entre [20, 200]
- saltos_cuerda_raw: Normal (μ=150, σ=40), limitado entre [50, 300]
- flexiones_60s_raw: Normal (μ=60, σ=15), limitado entre [20, 100]
- plancha_raw: Normal (μ=120, σ=40), limitado entre [30, 300]
- velocidad_reaccion_ms: Normal (μ=150, σ=30), limitado entre [80, 300]
- fuerza_impacto_psi: Normal (μ=1100, σ=200), limitado entre [800, 1500]

2.4 Scores Derivados:
---------------------
- equilibrio_score: Uniforme [1, 10]
- flexibilidad_score: Uniforme [1, 10]
- coordinacion_score: Uniforme [1, 10]

2.5 Normalización de Métricas (Escala 1-10):
---------------------------------------------
Fórmula general: valor_norm = (valor_crudo / valor_máximo) * 10

Fórmulas específicas:
- precision_jab_norm = (precision_jab_raw / 30) * 10
- combinacion_123_norm = (combinacion_123_raw / 30) * 10
- bloqueo_reflejo_norm = (bloqueo_reflejo_raw / 20) * 10
- resistencia_golpes_norm = (resistencia_golpes_raw / 150) * 10
- golpes_por_minuto_norm = (golpes_por_minuto_raw / 200) * 10
- saltos_cuerda_norm = (saltos_cuerda_raw / 300) * 10
- flexiones_60s_norm = (flexiones_60s_raw / 100) * 10
- plancha_norm = (plancha_raw / 300) * 10
- velocidad_reaccion_norm = (1 - (velocidad_reaccion_ms - 80) / 220) * 10
- fuerza_impacto_norm = (fuerza_impacto_psi / 1500) * 10

2.6 Macro-Atributos (Agrupación de Habilidades):
-----------------------------------------------
- Tecnica = promedio(precision_jab_norm, combinacion_123_norm, coordinacion_score)
- Potencia = promedio(fuerza_impacto_norm, golpes_por_minuto_norm)
- Resistencia = promedio(resistencia_golpes_norm, saltos_cuerda_norm, flexiones_60s_norm)
- Agilidad = promedio(velocidad_reaccion_norm, equilibrio_score, flexibilidad_score)

2.7 Métricas Físicas Calculadas:
--------------------------------
- imc = peso / (estatura^2)
- alcance_estimado = estatura * 1.05
- experiencia_nivel = anos_experiencia * (2 si nivel='pro' else 1)
- indice_atletico = promedio(tecnica, potencia, resistencia, agilidad)
- perfil_fisico = promedio(equilibrio_score, flexibilidad_score, coordinacion_score)

2.8 Historial de Combates:
---------------------------
- total_peleas: 
  * Profesionales: Exponencial (λ=20), mínimo 1
  * Amateurs: Exponencial (λ=10), mínimo 1
- ratio_victorias: Beta(α=2, β=1.5) [sesgo hacia más victorias]
- actividad_reciente: Poisson(λ=3) [combates en últimos 12 meses]
- actividad_reciente_norm: min(1.0, actividad_reciente / 10)

2.9 Sistema de Ranking:
-----------------------
- elo_base = 1500
- elo_variation = (indice_atletico - 5) * 100 + (anos_experiencia * 10)
- elo_rating = limitado entre [1000, 2500] con ruido Normal(μ=0, σ=100)
- ranking_score = 0.6 * indice_atletico + 0.3 * ratio_victorias * 10 + 0.1 * actividad_reciente_norm * 10
- ranking_position: Calculado tras ordenar por ranking_score descendente

3. REGLAS DE INTRODUCCIÓN DE RUIDO Y ANOMALÍAS
==============================================

3.1 Datos Faltantes:
--------------------
- 2% de valores nulos (np.nan) introducidos aleatoriamente en macro-atributos
- Probabilidad de celda faltante: [0.02, 0.98]

3.2 Simulación de Fatiga/Lesión:
--------------------------------
- 5% de atletas seleccionados aleatoriamente (sample(frac=0.05, random_state=42))
- Factor de reducción: Uniforme [0.70, 0.90] (reducción del 10-30%)
- Aplicado a:
  * Fuerza *= factor_reduccion
  * Resistencia *= factor_reduccion  
  * Velocidad *= factor_reduccion * 1.1 (menor impacto)

4. REGLAS DE GENERACIÓN DE COMBATES SINTÉTICOS
==============================================

4.1 Selección de Participantes:
-------------------------------
- Selección aleatoria de 2 boxeadores sin reemplazo
- IDs tomados del pool completo de boxeadores expandidos

4.2 Cálculo de Diferencias Entre Boxeadores:
--------------------------------------------
- diff_tecnica = boxer1['tecnica'] - boxer2['tecnica']
- diff_potencia = boxer1['potencia'] - boxer2['potencia']
- diff_resistencia = boxer1['resistencia'] - boxer2['resistencia']
- diff_agilidad = boxer1['agilidad'] - boxer2['agilidad']
- diff_ranking = boxer1['ranking_score'] - boxer2['ranking_score']
- diff_experiencia = boxer1['anos_experiencia'] - boxer2['anos_experiencia']
- diff_peso = boxer1['peso'] - boxer2['peso']
- diff_estatura = boxer1['estatura'] - boxer2['estatura']

4.3 Factores Contextuales:
--------------------------
- ventaja_local: [0, 1] con probabilidades [0.7, 0.3] (30% ventaja local)

4.4 Modelo de Predicción de Victoria:
------------------------------------
Fórmula de probabilidad base (para boxeador 1):
prob_base = 0.5 + diff_tecnica * 0.02 + diff_potencia * 0.02 + 
            diff_resistencia * 0.015 + diff_agilidad * 0.015 + 
            diff_ranking * 0.01 + diff_experiencia * 0.005 + 
            ventaja_local * 0.1

Limitaciones: prob_victoria_calculada ∈ [0.1, 0.9]

4.5 Determinación de Resultado:
------------------------------
- ganador: 1 si random() < prob_victoria_calculada, sino 2

4.6 Tipo de Victoria:
--------------------
Distribución de tipos de victoria:
- 'Decision': 60%
- 'KO': 15%  
- 'TKO': 10%
- 'Unanimous': 10%
- 'Split': 5%

4.7 Duración del Combate (Rounds):
----------------------------------
Para KO/TKO:
- Rounds 1-7 con probabilidades [0.3, 0.25, 0.2, 0.15, 0.05, 0.03, 0.02]

Para Decision/Unanimous/Split:
- 10 rounds: 70%
- 12 rounds: 30%

5. ESTRUCTURA FINAL DEL DATASET
===============================

5.1 Boxeadores (73 columnas):
-----------------------------
Identificación: id_atleta, nombre
Demográficos: edad, peso, estatura, guardia, nivel, division, pais, anos_experiencia
Métricas crudas: 10 variables de rendimiento físico
Métricas normalizadas: 10 variables correspondientes (escala 1-10)
Scores derivados: 3 variables (equilibrio, flexibilidad, coordinación)  
Macro-atributos: 4 variables principales (Tecnica, Potencia, Resistencia, Agilidad)
Métricas físicas: 5 variables calculadas
Historial: 4 variables de combates
Ranking: 3 variables de posición y score

5.2 Combates (13 columnas):
---------------------------
Identificación: combate_id, boxeador_1_id, boxeador_2_id
Diferencias: 8 variables de diferencias entre boxeadores
Contexto: ventaja_local
Resultado: ganador, tipo_victoria, rounds, prob_victoria_calculada

6. VALIDACIONES Y CONSISTENCIA
==============================

6.1 Validaciones Automáticas:
-----------------------------
- Todos los valores de edad están en rango [18, 45]
- Todos los pesos están en rango [47, 125] kg
- Todas las estaturas están en rango [1.55, 2.10] m
- Los macro-atributos están en escala [1, 10]
- Los ELO ratings están en rango [1000, 2500]
- Las probabilidades de victoria están en rango [0.1, 0.9]

6.2 Consistencia Lógica:
------------------------
- Los boxeadores profesionales tienen mayor experiencia promedio
- Los rankings se recalculan después de expansión
- Los IDs de combate son secuenciales y únicos
- No hay auto-enfrentamientos (boxeador vs sí mismo)

7. PARÁMETROS DE REPRODUCIBILIDAD
=================================

7.1 Seeds Utilizadas:
---------------------
- random.seed(42)
- np.random.seed(42)  
- Faker seed implícita

7.2 Versiones de Dependencias:
------------------------------
- pandas==2.0.3
- numpy==1.24.3
- faker==19.3.0
- uuid (librería estándar Python)

7.3 Archivos de Salida:
-----------------------
- data/boxeadores_ranking.csv (100,000 registros)
- data/combates_historicos.csv (50,000 registros)

NOTA: Todas estas reglas garantizan la generación de un dataset sintético 
realista, balanceado y adecuado para el entrenamiento de modelos de machine 
learning en el dominio del boxeo amateur y profesional.
