# Automatización RPA (Python) y Optimización de Sistemas Legacy

**Headline:** Desarrollo de un flujo de trabajo RPA para la carga masiva de datos y gestión documental en entornos sin API, logrando una reducción del 80% en tiempos operativos.

---

### 1. El Desafío (Contexto Operativo)
* **Cuello de Botella Manual:** La carga mensual de liquidaciones, impuestos y servicios en el sistema de gestión *legacy* (Inmosoft) requería aproximadamente 200 operaciones manuales unidad por unidad, consumiendo un promedio de 5 horas de trabajo[cite: 7].
* **Entorno Cerrado:** El sistema destino carecía de una API oficial o base de datos accesible, obligando a interactuar exclusivamente a través de su Interfaz Gráfica de Usuario (GUI)[cite: 7, 9].
* **Gestión Documental Ineficiente:** La descarga posterior de los cupones de pago (PDFs) requería su apertura manual para identificar a qué complejo pertenecían, propiciando un margen de error humano en la clasificación del 15% al 20%[cite: 8].

### 2. La Acción (Arquitectura RPA y Resiliencia)
Se diseñó un ecosistema de automatización en Python estructurado en tres fases (Transformación, Ingesta y Organización):

* **Data Pipeline (Excel a JSON):** Desarrollo de un script (`excel_a_json.py`) utilizando `pandas` para ingerir la matriz de liquidación financiera, validar columnas obligatorias y estructurarla en un JSON optimizado para el robot[cite: 7].
* **Motor de Automatización GUI:** Construcción del núcleo RPA (`liquidaciones_batch.py`) con `PyAutoGUI` y `PyGetWindow`. El sistema utiliza un archivo de calibración (`coordenadas_v6.json`) para orquestar los clics, tipeo y navegación dentro del software *legacy*[cite: 7, 9].
* **Trazabilidad y *Retry Logic*:** Para garantizar la estabilidad en una ejecución de larga duración (GUI Automation), se implementó un sistema de *logging* continuo y un mecanismo de *checkpoints* (`carga_checkpoint.json`). Si la automatización falla por un popup inesperado o lentitud del sistema, el script reanuda la operación exactamente desde la última unidad procesada exitosamente[cite: 9, 12].
* **Indexación y Enriquecimiento de PDFs:** Implementación de un flujo post-procesamiento que lee la base de datos maestra y renombra los PDFs descargados, añadiendo metadatos descriptivos en el título y moviéndolos a un árbol de directorios clasificado por complejo y condición (Inquilinos/Propietarios)[cite: 8].

### 3. El Impacto (Resultados)
* **Eficiencia Operativa:** El tiempo de procesamiento total de carga se redujo de 5 horas a menos de 1 hora (mejora del 80%), procesando exitosamente el 100% de los conceptos requeridos[cite: 7].
* **Mitigación de Riesgo:** La tasa de errores de tipeo y la omisión de unidades se redujeron a un 0% absoluto gracias a la lectura estructurada de datos[cite: 7].
* **Optimización de e-Discovery Interno:** El tiempo de clasificación e identificación de los 53 cupones de pago se redujo en un 99.7% (de 30 minutos a menos de 5 segundos), facilitando la auditoría visual instantánea[cite: 8].

---

### 💻 Fragmento Conceptual: Manejo de Estado y *Retry Logic*

```python
# Fragmento del motor RPA: Reanudación ante fallos de GUI
def guardar_checkpoint(unidad_codigo):
    """Guarda un checkpoint con la última unidad procesada exitosamente"""
    try:
        checkpoint_data = {
            'ultima_unidad': unidad_codigo,
            'timestamp': datetime.now().isoformat(),
            'estado': 'completado'
        }
        with open('carga_checkpoint.json', 'w', encoding='utf-8') as f:
            json.dump(checkpoint_data, f, indent=2)
            
        logger.info(f"🔄 Checkpoint guardado: última unidad = {unidad_codigo}")
    except Exception as e:
        logger.error(f"Error guardando checkpoint: {e}")

# [...] Bucle principal de ejecución RPA
for i in range(indice_inicio, len(unidades_lista)):
    unidad_nombre, conceptos = unidades_lista[i]
    
    try:
        if procesar_unidad(unidad_nombre, conceptos):
            exitos += 1
            logger.info(f"✅ Unidad {unidad_nombre} procesada exitosamente")
            guardar_checkpoint(unidad_nombre)
        else:
            errores += 1
            logger.error(f"❌ Unidad {unidad_nombre} falló al procesar")
    except Exception as e:
        logger.error(f"❌ Error fatal en {unidad_nombre}: {e}")
        # El checkpoint previo asegura que no se dupliquen cargas al reiniciar
        break
Stack Tecnológico:
Python (PyAutoGUI, Pandas, PyGetWindow) | RPA (Robotic Process Automation) | Sistemas Legacy | JSON | Audit Trails (Logging).
