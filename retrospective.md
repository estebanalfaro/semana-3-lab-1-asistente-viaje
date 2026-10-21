# Retrospectiva y Reflexión - Laboratorio de Diseño Técnico (SDD)

## 1. Paso de Requerimientos a Diseño
¿Qué decisiones técnicas (manejo de estado, división de componentes) tuvo que tomar la IA que no estaban escritas en el requirements.md?

* **Respuesta:**  En el requirments.md solo decía que la app debía recordar los viajes, entonces se decidió usar el almacenamiento interno del navegador (localStorage) y se creó la regla
de nunca guardar los datos calculados.

## 2. Memoria Semántica
¿Cómo afectó tener el archivo CLAUDE.md a la hora de que la IA propusiera los nombres de las funciones y componentes en el design.md?

* **Respuesta:**  Fue clave para mantener la consistencia, gracias a esto la IA no introdujo sinónimos genéricos sino que mantuvo los términos exactos exigidos por el cliente.

## 3. Poda Arquitectónica
¿Tuviste que simplificar alguna propuesta sobreestructurada de la IA para mantener el diseño simple y legible?

* **Respuesta:** Si, en las primeras iteraciones, la IA tendía a sobreestructurar la solución sugiriendo librerías de gestión de estado globales como Redux pesadas o persistencia en bases de
datos del navegador más complejas, por ende se forzó a la IA a simplificar y mantener la persistencia directa en localStorage