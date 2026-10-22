# Limitaciones conocidas del flujo automático (Sheets → Make → Claude)

Este documento resume lo que se detectó en las pruebas del flujo con la API. El revisor ya funciona bien en la mayoría de los casos, pero no es infalible. Léelo antes de publicar cualquier texto generado de forma automática.

## 1. Lo que el revisor todavía puede pasar por alto

En las pruebas, el revisor de la API dejó pasar estos casos:

- **Exageraciones de cercanía.** Frases como *"at your doorstep"* (o similares, como *"right outside your door"*) prometen una distancia que no está verificada. La regla de distancias solo permite *close to*, *near*, *nearby* y *close by*, pero el revisor no siempre marca estas expresiones figuradas.
- **Titulares sin el nombre del barrio.** El titular del anuncio debe incluir el vecindario (por ejemplo, *"... in North Park"*). El revisor a veces aprueba titulares que no lo mencionan.

**Qué hacer:** en la revisión humana, revisa siempre el titular y busca cualquier frase que exagere la cercanía a playas, parques o comercios.

## 2. El redactor a veces usa "walk"

Aunque la regla lo prohíbe, el redactor a veces escribe frases como *"walk to the beach"* o *"walkable"*. **El revisor sí lo detecta** y lo marca como corrección, así que el texto no sale aprobado con ese error. Aun así, ese texto tendrá que corregirse y volver a revisarse.

## 3. Cada ejecución genera un texto distinto

Claude no produce el mismo texto dos veces, aunque los datos de la propiedad sean idénticos. Eso significa que:

- Un texto puede salir perfecto en una ejecución y con errores en la siguiente.
- Que una prueba haya salido bien no garantiza que todas las siguientes salgan bien.

Por eso **la revisión humana final es obligatoria** antes de publicar cualquier anuncio, email o post, aunque el revisor haya dado APPROVED y PUBLISH-READY: YES.

## 4. Errores detectados en revisión humana (ya corregidos)

Casos que el revisor de la API aprobó, que encontró una persona al revisar el resultado y que ya se corrigieron en las reglas. Sirven de ejemplo de por qué la revisión humana sigue siendo obligatoria.

### Correo con "no dogs" sin la nota de animales de asistencia (octubre de 2026)

- **Qué pasó:** en una prueba en Make, el correo a interesados resumía la política de mascotas como *"no dogs"*, sin la nota *"(assistance animals welcome per applicable law)"*. El revisor lo aprobó.
- **Por qué:** la regla de la nota solo estaba escrita para los anuncios (*"if a listing says no pets…"*), así que ni el redactor ni el revisor la aplicaban al correo ni al post.
- **Riesgo:** es un problema de Fair Housing. Los animales de asistencia no son mascotas, y una restricción de mascotas sin la nota puede leerse como que también se excluyen.
- **Corrección:** en `referencias/marca.md`, `.claude/agents/redactor.md`, `.claude/agents/revisor.md` y los dos prompts de la API, la nota ahora es obligatoria en **cualquier** texto (anuncio, correo o post) que mencione una restricción de mascotas, incluso cuando el texto solo la resume. Si falta, el revisor lo marca como **CRITICAL** (Fair Housing) y el veredicto es REJECTED.
- **Qué revisar a mano:** busca *pet*, *dog*, *cat* y *animal* en los tres textos; toda restricción debe ir seguida de la nota.

## 5. Ideas de mejora a futuro

- **Verificación del titular.** Agregar un paso (o una instrucción específica) que compruebe que el titular incluye el nombre del barrio y el tipo de propiedad.
- **Lista fija de palabras prohibidas revisada por Make (sin IA).** Un módulo de Make (por ejemplo, un filtro de texto o una búsqueda con expresiones regulares) que busque palabras y frases prohibidas (*walk*, *steps from*, *at your doorstep*, *perfect for families*, etc.). Al no depender de la IA, da siempre el mismo resultado.
- **Reescritura automática cuando el veredicto no es APPROVED.** Si el revisor devuelve APPROVED WITH CHANGES o REJECTED, Make envía el texto y la tabla de correcciones de vuelta al redactor, y el nuevo texto pasa otra vez por el revisor (con un límite de intentos para no entrar en un bucle).
