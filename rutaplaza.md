---
equipo: 7
integrantes: Ruben David Zapata Bravo, Juan Carlos Escobar gil, Jaime leon ramirez Guarin
nivel: Pregrado
caso: Proyecto final pregrado
nombre_del_sistema: Ruta Plaza
formato_de_entrega: Widget
enlace_o_archivo_de_la_entrega: https://firenzy-tech.github.io/Publicar-Widget-Arquitectura/
---

# Entrega Bloque 2: escenarios de calidad y ADR (texto plano)

## 1. Caso y ficha de contexto

**Cliente, en su voz:**
“Tenemos un patio con 12 plazas de descarga y recibimos unos 80 camiones al día, entre las 6 a. m. y las 6 p. m., con el pico entre las 7 y las 10 de la mañana. Hoy los camiones esperan y se estorban entre sí. No queremos cambiar nuestros sistemas: queremos una API que nos diga a qué plaza y en qué franja va cada camión, que se ajuste sola cuando alguien llega tarde o una plaza queda fuera de servicio, y que baje la espera promedio en un 20 %. Y no puede caerse en plena hora pico.”

**Anclas con cifras (cinco):**
1. 12 plazas de descarga operativa.
2. ~80 camiones al día, de 6 a. m. a 6 p. m.
3. El webhook debe confirmar en ≤ 500 ms y la API replanificar en ≤ 2 s.
4. Bajar la espera promedio en un 20 %.
5. Venta única ~150 M COP + soporte hasta 1,5 M COP/mes (Costo base del proyecto 120 M).

**Tensión dominante, en una línea:**
En este sistema, la eficiencia de desempeño choca con la fiabilidad porque garantizar que no se pierda ni se duplique ningún evento (colas persistentes, reintentos) añade tiempo y recursos a cada solicitud, mientras el webhook debe confirmar en ≤ 500 ms y la API replanificar en ≤ 2 s.

**Dónde está esto en la entrega:**
Sección principal superior (Tarjetas "El cliente, en su voz" y "Tensión dominante", y sección "2. La economía").

---

## 2. Supuestos declarados 

[FALTA] (No aplica para nivel pregrado según la entrega).

---

## 3. Priorización con presupuesto de 18 puntos

- Adecuación funcional: 1 | Justificación: Somos una librería con casos de uso casi infinitos y solo ofrecemos un algoritmo reutilizable; la adecuación funcional debe venir de quienes la usen, no de nosotros.
- Eficiencia de desempeño: 4 | Justificación: La API debe operar a su máximo rendimiento para calcular las mejores rutas.
- Compatibilidad: 1 | Justificación: Se usará con un único protocolo, REST.
- Usabilidad: 1 | Justificación: Solo se espera que la usen profesionales, no usuarios distintos.
- Fiabilidad: 4 | Justificación: El sistema debe mantener los datos confiables y estar disponible en distintos escenarios.
- Seguridad: 3 | Justificación: Queremos garantizar que personas externas no puedan acceder al sistema.
- Mantenibilidad: 3 | Justificación: Queremos que la API pueda crecer (escalar) con el tiempo.
- Portabilidad: 1 | Justificación: Como es una API, quien la use debe adaptarse a ella y no al revés.

**Total:** 18

**Qué bajamos para poder subir el atributo más alto, y qué riesgo aceptamos:**
Para subir desempeño y fiabilidad a 4 bajamos a 1 la adecuación funcional, la compatibilidad, la usabilidad y la portabilidad. Aceptamos que el cliente tendrá que construir sobre la API lo específico de su operación, que un cliente que no hable REST necesitará un adaptador, que una integración o documentación difícil afectará a quien no sea profesional y que mover la API a otra plataforma costará más.

**Dónde está esto en la entrega:**
Sección "3. Dónde pusimos el esfuerzo".

---

## 4. Escenarios de calidad

### E1
- Atributo dominante: Eficiencia de desempeño
- Otro atributo que toca (si lo hay): Fiabilidad (la medida exige 0 eventos perdidos).
- Fuente del estímulo: Saturación de recepción de carga.
- Estímulo: Alta temporada de entrada y salida de carga por tanto el sistema presenta ralentización y fallos.
- Artefacto: Lectura de Datos.
- Entorno: Durante los meses previos a temporadas navideñas.
- Respuesta: La API usa webhook para garantizar la integración, procesando la inserción de datos sin abusar de recursos y aprovechando periodos de baja demanda.
- Medida de la respuesta: Carga pico de 20 sol/min (doble del normal) por 3h: lectura p95 ≤ 800ms, p99 ≤ 2s; confirmación webhook ≤ 500ms; proc. sostenido 20 evt/min sin degradación; error ≤ 1%; 0 perdidos; CPU ≤ 70%, RAM ≤ 80%.
- ¿Es crítico? (sí | no) y por qué: Sí. Eficiencia de desempeño tiene 4 puntos y la opción que mejor la cumple cuesta mantenibilidad y compatibilidad.
- Dónde está en la entrega: Sección "5. Decisiones Arquitectónicas (ADR - Bloque 2)", tarjeta ADR-01.

### E2
- Atributo dominante: Fiabilidad
- Otro atributo que toca (si lo hay): Eficiencia de desempeño (persistir y reintentar añade tiempo a cada evento).
- Fuente del estímulo: Falla del servicio que aloja la API.
- Estímulo: En plena hora pico de llegada de camiones, el servicio se cae o se reinicia mientras el webhook sigue recibiendo eventos.
- Artefacto: Módulo de recepción de eventos (webhook).
- Entorno: Apagón, en horario laboral de 6 am a 5 pm.
- Respuesta: Guarda en cola persistente para no perderlos, se recupera automáticamente, procesa sin duplicarlos (ID único), sigue entregando el último plan válido y notifica la actualización al volver.
- Medida de la respuesta: 0 perdidos; 0 duplicados procesados; recup. automática ≤ 60s; disp. ≥ 99.5% en horario operativo; ≥ 99% de reintentos exitosos en máx 3 intentos.
- ¿Es crítico? (sí | no) y por qué: Sí. Fiabilidad tiene 4 puntos y garantizar 0 perdidos cuesta dinero, seguridad y desempeño.
- Dónde está en la entrega: Sección "5. Decisiones Arquitectónicas (ADR - Bloque 2)", tarjeta ADR-02.

### E3
- Atributo dominante: Mantenibilidad
- Otro atributo que toca (si lo hay): Eficiencia de desempeño y fiabilidad (leer reglas externas añade latencia y puede corromper la asignación).
- Fuente del estímulo: Cliente que ajusta la operación del predio.
- Estímulo: Se agrega una nueva plaza de descarga y una nueva regla (ej. material que solo puede descargarse en ciertas plazas y horarios).
- Artefacto: Módulos de asignación y validación de reglas de patio.
- Entorno: Tiempo de desarrollo y actualización.
- Respuesta: Se modifica el código aislando el impacto, ejecutando pruebas automáticas para confirmar que la lógica existente no se rompió.
- Medida de la respuesta: Cambio completado en ≤ 2 días; modifica ≤ 2 módulos; 100% de pruebas pasan (0 regresiones) + nuevas pruebas; cobertura ≥ 80%; despliegue en ≤ 1 hora sin inactividad.
- ¿Es crítico? (sí | no) y por qué: Sí. Mantenibilidad tiene 3 puntos y las opciones que más la favorecen cuestan desempeño o fiabilidad, que valen 4.
- Dónde está en la entrega: Sección "5. Decisiones Arquitectónicas (ADR - Bloque 2)", tarjeta ADR-03.

---

## 5. Selección de escenarios críticos

**Escenarios críticos elegidos:** E1, E2 y E3.

**Por qué esos y no los demás (una línea):**
Son los dos atributos con 4 puntos (desempeño y fiabilidad) y mantenibilidad (3), y en los tres cumplir la medida cuesta otro atributo importante. Los atributos con 1 punto no son críticos porque quien usa la API se adapta a ella.

---

## 6. ADR (uno por cada escenario crítico)

### ADR-1
- Título: Gestión de picos de carga en el Webhook
- Estado: Propuesto
- Escenario que lo origina: E1
- Context: sí

**Opción A:**
- Qué haría el sistema: Procesamiento síncrono con base de datos en clúster.
- Gana: Arquitectura simple sin componentes adicionales.
- Paga: Eficiencia de desempeño (el tiempo de respuesta del webhook dependerá de la BD, arriesgando la medida de confirmación ≤ 500ms).
- Atributo que sacrifica: Eficiencia de desempeño.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: No. Porque bajo picos el tiempo de respuesta se degradará superando los 500ms exigidos.
- Decisión de producto o de política que encierra: Favorece la simplicidad operativa de la infraestructura sacrificando el rendimiento en picos.

**Opción B:**
- Qué haría el sistema: Cola de mensajes externa (ej. Broker Pub/Sub) y Workers asíncronos.
- Gana: Confirmación ultrarrápida al webhook, aislando los picos de carga.
- Paga: Mantenibilidad y Compatibilidad (introduce un nuevo punto de fallo y mayor complejidad de despliegue).
- Atributo que sacrifica: Mantenibilidad y Compatibilidad.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: Sí. Porque aislar la recepción de los datos garantiza que el webhook responda en menos de 500ms.
- Decisión de producto o de política que encierra: Favorece el rendimiento técnico trasladando la carga financiera al costo operativo de mantener un broker en la nube.

**Opción C:** 
- Qué haría el sistema: Inserción en memoria (Buffer) y volcado periódico (Batching).
- Gana: Menor consumo de conexiones a base de datos.
- Paga: Fiabilidad (si el sistema se reinicia antes del volcado, se incumpliría la métrica de 0 eventos perdidos).
- Atributo que sacrifica: Fiabilidad.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: Depende (Cumple parcialmente). Cumple en velocidad pero falla en la métrica de 0 eventos perdidos si hay una caída repentina.
- Decisión de producto o de política que encierra: Favorece la economía de recursos asumiendo un riesgo inaceptable sobre los datos del cliente.

**Lo que nos falta saber para elegir en el Bloque 3:**
Necesitamos definir si el presupuesto de 120 M soporta los costos recurrentes de infraestructura (nube) que implican las colas de mensajes.

**Dónde está este ADR en la entrega:**
Sección "5. Decisiones Arquitectónicas (ADR - Bloque 2)", tarjeta ADR-01.

### ADR-2
- Título: Persistencia de eventos ante caída del servicio
- Estado: Propuesto
- Escenario que lo origina: E2
- Context: sí

**Opción A:**
- Qué haría el sistema: Almacenamiento local temporal (Log en disco) en el nodo.
- Gana: Fiabilidad básica sin acoplamiento a servicios de terceros.
- Paga: Portabilidad y Fiabilidad estricta (si el nodo físico se destruye o cambia de instancia en la nube, los eventos en disco se pierden).
- Atributo que sacrifica: Portabilidad y Fiabilidad estricta.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: Depende (Cumple parcialmente). Si la instancia se destruye físicamente no cumple el margen de 0 eventos perdidos.
- Decisión de producto o de política que encierra: Traslada el riesgo de pérdida a la estabilidad física del nodo que aloja la API.

**Opción B:**
- Qué haría el sistema: Delegación de reintentos al cliente (Failing Fast).
- Gana: Eficiencia de desempeño y mantenibilidad al no gestionar colas internamente.
- Paga: Adecuación funcional y Usabilidad (se obliga al sistema origen a hacerse cargo del buffer, violando la premisa de 0 eventos perdidos por parte nuestra).
- Atributo que sacrifica: Adecuación funcional y Usabilidad.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: No. Porque viola explícitamente nuestra premisa de 0 eventos perdidos.
- Decisión de producto o de política que encierra: Traslada la complejidad operativa y la responsabilidad técnica del reintento al cliente.

**Opción C:** 
- Qué haría el sistema: Broker de mensajería externo con persistencia nativa garantizada.
- Gana: Recuperación automática delegada y garantía de orden/no duplicación.
- Paga: Costo operativo y Seguridad (mayor superficie de ataque al exponer un nuevo canal de comunicación).
- Atributo que sacrifica: Costo operativo y Seguridad.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: Sí. Los brokers están diseñados para garantizar los 0 perdidos, 0 duplicados y recuperación inmediata.
- Decisión de producto o de política que encierra: Favorece la resiliencia y fiabilidad extrema trasladando el costo a licenciamiento o servicios administrados adicionales.

**Lo que nos falta saber para elegir en el Bloque 3:**
Necesitamos saber si las políticas de seguridad del cliente permiten desplegar y exponer un canal de mensajería externo en su red.

**Dónde está este ADR en la entrega:**
Sección "5. Decisiones Arquitectónicas (ADR - Bloque 2)", tarjeta ADR-02.

### ADR-3
- Título: Desacoplamiento de reglas de patio
- Estado: Propuesto
- Escenario que lo origina: E3
- Context: sí

**Opción A:**
- Qué haría el sistema: Patrón Strategy incrustado en código (Reglas inyectadas).
- Gana: Alta cohesión y control absoluto mediante pruebas unitarias en código (asegura el 100% de pruebas sin regresiones).
- Paga: Cada nueva regla requiere un ciclo completo de CI/CD para el despliegue de la API completa.
- Atributo que sacrifica: Mantenibilidad (agilidad de despliegue a largo plazo).
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: No. Porque a largo plazo cada regla nueva requeriría tocar código fuente y dificultaría cumplir el despliegue en ≤ 1 hora.
- Decisión de producto o de política que encierra: Mantiene el control total en el equipo de desarrollo, trasladando la lentitud del proceso al cliente.

**Opción B:**
- Qué haría el sistema: Motor de reglas de negocio externo (Business Rules Engine).
- Gana: Mantenibilidad superlativa (las reglas se cambian sin tocar el código fuente, eliminando tiempos de despliegue).
- Paga: Eficiencia de desempeño (evaluar políticas vía red añade latencia constante por cada camión asignado).
- Atributo que sacrifica: Eficiencia de desempeño.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: Sí. Cumple con la rapidez de los cambios, asumiendo el impacto moderado en la latencia.
- Decisión de producto o de política que encierra: Favorece la agilidad operativa del negocio asumiendo un costo técnico en rendimiento (latencia).

**Opción C:** 
- Qué haría el sistema: Tablas paramétricas en Base de Datos leídas en caliente.
- Gana: Flexibilidad para configurar horarios/plazas desde una interfaz externa.
- Paga: Seguridad y Fiabilidad (las configuraciones erróneas en base de datos pueden eludir las pruebas automáticas exigidas en la métrica y corromper la asignación).
- Atributo que sacrifica: Seguridad y Fiabilidad.
- ¿Cumple la medida de la respuesta? (sí | no | depende) y por qué: Depende (Cumple parcialmente). Depende de cuán estrictas sean las validaciones de negocio implementadas antes de guardar los datos en las tablas.
- Decisión de producto o de política que encierra: Favorece una solución de bajo costo y fácil implementación, sacrificando la seguridad contra errores humanos al configurar.

**Lo que nos falta saber para elegir en el Bloque 3:**
Conocer con qué frecuencia real el cliente ajusta estas políticas para evaluar si la complejidad del motor de reglas vale la pena.

**Dónde está este ADR en la entrega:**
Sección "5. Decisiones Arquitectónicas (ADR - Bloque 2)", tarjeta ADR-03.

---

## 7. Lo que el equipo reconoce

**Puntos débiles de nuestra entrega que ya identificamos:**
[FALTA]

**Lo que más discutimos como equipo y no resolvimos:**
[FALTA]

---

## 8. Autochequeo del equipo (marque lo que se cumple)

- [x] Las anclas tienen cifras, no solo adjetivos.
- [x] Podemos decir en una línea cuál es la tensión dominante de nuestro caso.
- [ ] Declaramos al menos cinco supuestos, cada uno con su razón (solo posgrado).
- [x] Nuestra priorización suma exactamente 18 y respeta que ningún atributo vale menos de 1 ni más de 5.
- [x] Cada escenario tiene estímulo observable, respuesta concreta y medida verificable.
- [x] Los ADR corresponden a escenarios críticos y justificamos por qué esos.
- [x] Cada ADR tiene al menos tres opciones, descritas como mecanismos y no como marcas (posgrado) o al menos dos (pregrado).
- [x] No elegimos todavía: ninguna Decision está escrita.

---

## 9. Inconsistencias detectadas por la IA (solo la IA diligencia; no corrija lo anterior)

- Ninguna detectada. Todo el contenido parece cuadrar correctamente con los puntajes, priorizaciones (suman 18) y restricciones solicitadas para este bloque.