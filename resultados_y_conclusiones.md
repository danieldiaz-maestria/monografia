
--- Notas Técnicas del Proyecto UWB ---
Fecha: 20 de enero de 2026
Autor: GitHub Copilot (basado en el TFG)


--- RESULTADOS CLAVE (LO QUE ENCONTRAMOS) ---

1.  **El "Body Shadowing" es el principal problema:** Confirmado. Cuando el cuerpo se interpone entre los dispositivos UWB, el error en la medición de distancia se dispara de forma dramática. Pasamos de una precisión excelente de ~5 cm en condiciones ideales (LOS - Línea de Visión Directa) a errores promedio de más de 80-90 cm en condiciones de bloqueo (NLOS - Sin Línea de Visión). En casos extremos, los picos de error llegaron a casi 3 metros.

2.  **La ubicación del sensor lo es TODO:** El lugar donde se lleva el dispositivo tiene un impacto masivo. Aquí un ranking rápido de mejor a peor rendimiento:
    *   **CAMPEONES (Más estables):** Cabeza y Muñeca. La cabeza, por su forma esférica, ayuda a que la señal se difracte (se "curve") a su alrededor. La muñeca, al estar en una extremidad móvil, no sufre un bloqueo tan constante.
    *   **DECENTES (Pero variables):** Mano. Similar a la muñeca, pero un poco más irregular.
    *   **PROBLEMÁTICOS (Errores altos):** Rodilla y Tobillo. La cercanía al suelo introduce muchos rebotes de señal (multitrayecto) que complican la medición.
    *   **LOS PEORES (Muy imprecisos en NLOS):** Cadera y Pecho. Son un "muro" para la señal. Aunque en LOS funcionan genial, en cuanto el cuerpo bloquea, el error se vuelve altísimo y muy variable. Hay que evitar estas zonas si se necesita precisión constante.

3.  **La paradoja del pasillo: a veces, los rebotes ayudan.** Contrario a lo que se podría pensar, en ciertas situaciones NLOS (especialmente con sensores en cabeza o muñeca), el sistema funcionó MEJOR en el pasillo interior que en el campo abierto. Esto se debe al "multitrayecto constructivo": las señales que rebotan en las paredes pueden encontrar un camino alternativo hacia el receptor, compensando el bloqueo del cuerpo. En el exterior, si se bloquea la señal, no hay nada que la "rescate".

4.  **El error no se comporta "normalmente":** Estadísticamente hablando, el error no sigue una distribución gaussiana (la típica campana de Gauss). En NLOS, la distribución tiene una "cola larga y pesada", lo que en la práctica significa que, aunque el error promedio sea uno, hay una alta probabilidad de que ocurran errores muy, muy grandes (los "outliers"). Esto es vital para diseñar algoritmos, ya que los filtros simples que asumen un error "normal" no funcionarán bien.


--- CONCLUSIONES PRINCIPALES (¿QUÉ SIGNIFICA TODO ESTO?) ---

1.  **Viabilidad confirmada, pero con condiciones:** La tecnología UWB a 6.5 GHz es totalmente viable para localización en interiores con precisiones que rondan los 50-90 cm en escenarios realistas. Esto es más que suficiente para muchas aplicaciones (logística, seguridad industrial, monitoreo de pacientes). Sin embargo, la idea de una precisión "mágica" de pocos centímetros solo es cierta en condiciones de laboratorio (LOS). El Body Shadowing es un factor de diseño ineludible.

2.  **Recomendaciones prácticas de diseño:**
    *   **Ubicación:** Si la precisión es crítica, el dispositivo debe ir en la cabeza o la muñeca. Si se tolera un error de ~1 metro y la comodidad es más importante, el pecho o la cadera son aceptables.
    *   **Algoritmos:** No se puede usar un simple cálculo de trilateración y esperar buenos resultados. Es necesario implementar filtros más avanzados (como un Filtro de Kalman, que usaste en la fase 2) que puedan manejar los picos de error y la naturaleza no gaussiana del mismo.
    *   **Detección NLOS:** Una de las claves es que el sistema sea capaz de "saber" cuándo una medición no es fiable (porque está en NLOS). Descubrimos que la variabilidad del tiempo de vuelo (ToF) es un gran indicador. Si las mediciones de tiempo empiezan a "bailar" mucho, es muy probable que haya un bloqueo. Un sistema robusto debería detectar esto y darle menos peso a esa medición.

3.  **El futuro es la fusión de sensores:** Para dar el siguiente salto en precisión, el camino es combinar UWB con otros sensores. Integrar una IMU (Unidad de Medición Inercial - como la que tienen los móviles) permitiría saber la orientación del cuerpo en tiempo real. Si el sistema sabe que "el usuario está de espaldas al ancla 3", puede aplicar un modelo de corrección específico para esa situación o, directamente, ignorar la medición de ese ancla.

4.  **No todas las personas son iguales:** Se notó una variabilidad importante en los resultados entre diferentes personas, incluso llevando el sensor en el mismo sitio. Características como la complexión física influyen. Un sistema de muy alta precisión podría requerir una breve "calibración" por usuario.
