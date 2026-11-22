# Cuentas y acceso

## Purpose

Permitir que una persona cree una cuenta en FlowSync, inicie y cierre sesión, y consulte su perfil, de modo que solo quien tiene una sesión válida accede a las pantallas privadas de la aplicación.

## Requirements

### Requirement: Registro de cuenta

La API SHALL crear una cuenta nueva a partir de nombre completo (opcional), email, contraseña y confirmación de contraseña, y SHALL devolver en la misma respuesta los datos públicos del usuario y un token de acceso, de modo que la persona queda con la sesión iniciada sin tener que hacer login.

#### Scenario: Registro correcto
- **WHEN** se envía una petición de registro con un email válido no registrado, una contraseña de entre 8 y 32 caracteres, la misma contraseña como confirmación y el nombre completo (o `null`)
- **THEN** la API responde con estado 200 y un cuerpo `data` que contiene `user` (identificador, nombre completo, email, fechas de creación y actualización, iniciales) y `token`

#### Scenario: Registro sin nombre
- **WHEN** se envía una petición de registro válida con el nombre completo a `null` o como texto vacío
- **THEN** la cuenta se crea y el usuario devuelto tiene el nombre completo a `null`

#### Scenario: Espacios alrededor del nombre
- **WHEN** se envía una petición de registro válida con espacios al principio o al final del nombre completo
- **THEN** la cuenta se crea con el nombre sin esos espacios

#### Scenario: Email que solo difiere en mayúsculas
- **WHEN** se envía una petición de registro con un email que coincide con el de una cuenta existente salvo en mayúsculas/minúsculas
- **THEN** la API crea una cuenta nueva e independiente

### Requirement: Validación del registro

La API SHALL rechazar con estado 422 cualquier registro que no cumpla las reglas de los datos, y SHALL indicar en cada error el campo afectado y la regla incumplida.

#### Scenario: Email ya registrado
- **WHEN** se envía una petición de registro con un email que ya pertenece a otra cuenta
- **THEN** la API responde 422 con un error sobre el campo `email` y la regla de unicidad, y no crea ninguna cuenta

#### Scenario: Email con formato inválido o demasiado largo
- **WHEN** se envía una petición de registro con un email sin formato de email o de más de 254 caracteres
- **THEN** la API responde 422 con un error sobre el campo `email`

#### Scenario: Contraseña fuera de rango
- **WHEN** se envía una petición de registro con una contraseña (y su confirmación) de menos de 8 o más de 32 caracteres
- **THEN** la API responde 422 con un error de longitud sobre el campo `password` y otro sobre `passwordConfirmation`

#### Scenario: Confirmación distinta
- **WHEN** se envía una petición de registro cuya confirmación de contraseña no coincide con la contraseña
- **THEN** la API responde 422 con un error sobre el campo `passwordConfirmation`

#### Scenario: Campo de nombre ausente
- **WHEN** se envía una petición de registro que no incluye el campo de nombre completo (ni siquiera a `null`)
- **THEN** la API responde 422 con un error de campo requerido sobre `fullName`

### Requirement: Inicio de sesión

La API SHALL emitir un token de acceso nuevo cuando el email y la contraseña corresponden a una cuenta existente, y SHALL devolverlo junto con los datos públicos del usuario.

#### Scenario: Credenciales correctas
- **WHEN** se envía una petición de login con el email y la contraseña de una cuenta existente
- **THEN** la API responde 200 con un cuerpo `data` que contiene `user` y un `token` nuevo

#### Scenario: Credenciales incorrectas
- **WHEN** se envía una petición de login con un email que no existe o con una contraseña que no corresponde a la cuenta
- **THEN** la API responde 400 con un único error genérico de credenciales inválidas, sin indicar qué campo falló

#### Scenario: Email con distinto uso de mayúsculas
- **WHEN** se envía una petición de login con el email de una cuenta existente escrito con otras mayúsculas/minúsculas y la contraseña correcta
- **THEN** la API responde 400 como credenciales incorrectas

#### Scenario: Email con formato inválido
- **WHEN** se envía una petición de login con un email sin formato de email
- **THEN** la API responde 422 con un error sobre el campo `email`

#### Scenario: Contraseña ausente
- **WHEN** se envía una petición de login sin el campo de contraseña
- **THEN** la API responde 422 con un error de campo requerido sobre `password`

#### Scenario: Contraseña de cualquier longitud
- **WHEN** se envía una petición de login con una contraseña de más de 32 caracteres
- **THEN** la API no la rechaza por longitud y responde como a cualquier otra credencial (400 si no corresponde a la cuenta)

#### Scenario: Varias sesiones a la vez
- **WHEN** la misma cuenta inicia sesión varias veces
- **THEN** cada inicio de sesión devuelve un token distinto y todos siguen siendo válidos a la vez

### Requirement: Consulta del perfil

La API SHALL devolver los datos públicos del usuario dueño del token cuando la petición incluye un token de acceso válido.

#### Scenario: Perfil con token válido
- **WHEN** se pide el perfil enviando un token de acceso válido como `Bearer`
- **THEN** la API responde 200 con un cuerpo `data` con identificador, nombre completo, email, fechas de creación y actualización e iniciales del usuario

### Requirement: Acceso protegido

La API SHALL rechazar con estado 401 las peticiones a operaciones de cuenta (perfil y cierre de sesión) que no incluyan un token de acceso válido.

#### Scenario: Sin token
- **WHEN** se pide el perfil o el cierre de sesión sin token de acceso
- **THEN** la API responde 401 con un error de acceso no autorizado

#### Scenario: Token inventado
- **WHEN** se pide el perfil con un token que la API nunca emitió
- **THEN** la API responde 401

#### Scenario: Token invalidado
- **WHEN** se pide el perfil con un token que ya se usó para cerrar sesión
- **THEN** la API responde 401

### Requirement: Cierre de sesión en la API

La API SHALL invalidar el token de acceso con el que se hace la petición de cierre de sesión, de modo que ese token deje de servir para operaciones posteriores.

#### Scenario: Logout correcto
- **WHEN** se envía una petición de cierre de sesión con un token válido
- **THEN** la API responde 200 con un mensaje de confirmación (sin envoltorio `data`) y ese token deja de ser aceptado

#### Scenario: Otras sesiones no se cierran
- **WHEN** una cuenta tiene varios tokens activos y cierra sesión con uno de ellos
- **THEN** solo ese token deja de ser aceptado; los demás siguen siendo válidos

### Requirement: Iniciales del usuario

La API SHALL incluir en los datos públicos del usuario unas iniciales de dos letras en mayúsculas.

#### Scenario: Nombre y apellido
- **WHEN** el usuario tiene un nombre completo de al menos dos palabras separadas por un único espacio
- **THEN** las iniciales son la primera letra de la primera palabra y la primera letra de la segunda

#### Scenario: Palabras separadas por varios espacios
- **WHEN** el usuario tiene un nombre completo cuyas dos primeras palabras están separadas por más de un espacio
- **THEN** las iniciales son las dos primeras letras de la primera palabra

#### Scenario: Nombre de una sola palabra
- **WHEN** el usuario tiene un nombre completo de una sola palabra
- **THEN** las iniciales son las dos primeras letras de esa palabra

#### Scenario: Sin nombre
- **WHEN** el usuario no tiene nombre completo
- **THEN** las iniciales son la primera letra de la parte del email anterior a la `@` y la primera letra de la parte posterior

### Requirement: Pantalla de registro

El frontend SHALL ofrecer una pantalla de registro con nombre completo (marcado como opcional), email, contraseña y repetición de contraseña, y SHALL dejar a la persona con la sesión iniciada en su perfil cuando el registro tiene éxito.

#### Scenario: Registro correcto desde la pantalla
- **WHEN** la persona, sin sesión, rellena el formulario con datos válidos y pulsa "Crear cuenta"
- **THEN** el botón muestra "Creando cuenta…" y queda deshabilitado mientras se procesa, y al terminar la persona ve su perfil

#### Scenario: Contraseñas que no coinciden
- **WHEN** la persona escribe dos contraseñas distintas y pulsa "Crear cuenta"
- **THEN** ve "Las contraseñas no coinciden." bajo el campo de repetición, sin que se envíe nada al servidor

#### Scenario: Errores de validación del servidor
- **WHEN** el servidor rechaza el registro por un error de validación de un campo del formulario
- **THEN** la persona ve un mensaje en español bajo ese campo (por ejemplo, "Ese email ya está registrado. Inicia sesión en su lugar.")

#### Scenario: Nombre en blanco
- **WHEN** la persona deja el nombre vacío o solo con espacios
- **THEN** la cuenta se crea sin nombre

#### Scenario: Enlace a login
- **WHEN** la persona pulsa "Inicia sesión" en la pantalla de registro
- **THEN** ve la pantalla de inicio de sesión

### Requirement: Pantalla de inicio de sesión

El frontend SHALL ofrecer una pantalla de inicio de sesión con email y contraseña, y SHALL llevar a la persona a su perfil cuando las credenciales son correctas.

#### Scenario: Login correcto desde la pantalla
- **WHEN** la persona, sin sesión, introduce credenciales correctas y pulsa "Entrar"
- **THEN** el botón muestra "Entrando…" y queda deshabilitado mientras se procesa, y al terminar ve su perfil

#### Scenario: Credenciales incorrectas
- **WHEN** la persona introduce credenciales incorrectas y pulsa "Entrar"
- **THEN** ve un aviso destacado "El email o la contraseña no son correctos."

#### Scenario: Servidor no disponible
- **WHEN** la persona intenta entrar y el servidor no responde
- **THEN** ve un aviso indicando que no se pudo conectar con el servidor

#### Scenario: Enlace a registro
- **WHEN** la persona pulsa "Crea una" en la pantalla de login
- **THEN** ve la pantalla de registro

### Requirement: Protección de pantallas

El frontend SHALL mostrar el perfil solo a personas con sesión válida, y SHALL mostrar las pantallas de login y registro solo a personas sin sesión.

#### Scenario: Perfil sin sesión
- **WHEN** una persona sin sesión abre la pantalla de perfil
- **THEN** es redirigida a la pantalla de inicio de sesión

#### Scenario: Login o registro con sesión
- **WHEN** una persona con sesión válida abre la pantalla de login o la de registro
- **THEN** es redirigida a su perfil

#### Scenario: Dirección desconocida
- **WHEN** cualquier persona abre una dirección que no corresponde a ninguna pantalla
- **THEN** es redirigida al perfil (y, si no tiene sesión, de ahí al login)

#### Scenario: Comprobación en curso
- **WHEN** la aplicación está comprobando una sesión guardada
- **THEN** la persona ve una pantalla de carga en lugar del perfil, el login o el registro

### Requirement: Persistencia de la sesión

El frontend SHALL conservar la sesión entre recargas de la página y SHALL validarla con el servidor al cargar la aplicación antes de darla por buena.

#### Scenario: Recarga con sesión válida
- **WHEN** una persona con sesión recarga la página o vuelve a abrir la aplicación en el mismo navegador
- **THEN** sigue con la sesión iniciada y ve su perfil sin volver a introducir credenciales

#### Scenario: Sesión rechazada por el servidor
- **WHEN** la aplicación carga con una sesión guardada que el servidor ya no acepta
- **THEN** la sesión guardada se descarta y la persona ve el login con el aviso "Tu sesión ha caducado. Vuelve a iniciar sesión."

#### Scenario: Servidor caído al cargar
- **WHEN** la aplicación carga con una sesión guardada y el servidor no responde o falla
- **THEN** la persona ve el login con un aviso del problema, y la sesión guardada se conserva para que funcione al recargar cuando el servidor vuelva

### Requirement: Pantalla de perfil

El frontend SHALL mostrar a la persona con sesión sus iniciales, su nombre completo, su email y la fecha en que se registró.

#### Scenario: Perfil con nombre
- **WHEN** una persona con sesión y nombre completo abre su perfil
- **THEN** ve sus iniciales en un círculo, su nombre, su email y "Miembro desde" con la fecha de alta en formato largo en español

#### Scenario: Perfil sin nombre
- **WHEN** una persona con sesión y sin nombre completo abre su perfil
- **THEN** ve "Sin nombre" en lugar del nombre

### Requirement: Cierre de sesión desde la pantalla

El frontend SHALL cerrar la sesión de la persona en el navegador cuando pulsa "Cerrar sesión", aunque el servidor no confirme el cierre.

#### Scenario: Cerrar sesión
- **WHEN** una persona con sesión pulsa "Cerrar sesión" en su perfil
- **THEN** ve inmediatamente la pantalla de inicio de sesión, sin aviso, y al recargar sigue sin sesión

#### Scenario: Cierre con el servidor caído
- **WHEN** una persona pulsa "Cerrar sesión" y el servidor no responde
- **THEN** la sesión igualmente se cierra en el navegador y la persona ve la pantalla de inicio de sesión

---

## Parte B

### 1. Requisitos generados y comprobados

- Requisitos escritos por el agente: **13** (46 escenarios tras la revisión; 38 en la primera versión)
- Requisitos comprobados por mí (Laura) consultando el código: **0**
- Requisitos comprobados por el agente a petición mía ("VALIDA TU"): **13**
  - Los 7 de API: leyendo el código y con peticiones reales contra el backend local.
  - Los 6 de pantallas: solo leyendo el código; ninguno se probó en el navegador.

La comprobación del agente cambió 7 de los 13 requisitos: 1 escenario era falso ("Cerrando sesión…" nunca se ve), 3 eran imprecisos (iniciales con nombre y apellido, contraseña fuera de rango, registro sin nombre) y se añadieron 8 escenarios que la primera versión no recogía (email en mayúsculas, espacios alrededor del nombre, varios espacios entre palabras, contraseña de login sin límite, contraseña ausente, varias sesiones, cerrar una sola sesión, token inventado).

| # | Requisito | Cómo se comprobó |
|---|---|---|
| 1 | Registro de cuenta | código + petición real |
| 2 | Validación del registro | código + petición real |
| 3 | Inicio de sesión | código + petición real |
| 4 | Consulta del perfil | código + petición real |
| 5 | Acceso protegido | código + petición real |
| 6 | Cierre de sesión en la API | código + petición real |
| 7 | Iniciales del usuario | código + petición real |
| 8 | Pantalla de registro | solo código |
| 9 | Pantalla de inicio de sesión | solo código |
| 10 | Protección de pantallas | solo código |
| 11 | Persistencia de la sesión | solo código |
| 12 | Pantalla de perfil | solo código |
| 13 | Cierre de sesión desde la pantalla | solo código |

### 2. Incoherencias detectadas

- Longitud de la contraseña: el registro exige 8–32 caracteres y el login acepta cualquier longitud (API, registro vs. login).
- Nombre completo: la pantalla de registro lo presenta como "opcional", pero la API responde 422 si el campo no viaja en la petición; solo su valor es opcional, no su presencia (pantalla de registro vs. API).
- Envoltorio de respuestas: registro, login y perfil responden dentro de `data`; el cierre de sesión responde `{ message }` sin `data` (API).
- Código de estado: el registro crea una cuenta pero responde 200, no 201 (API).
- Iniciales: con nombre y apellido se toma la inicial de cada palabra, pero si hay dos espacios entre ellas se toman las dos primeras letras del nombre ("Ana  López" → "AN"). Los espacios del principio y del final sí se limpian, los de en medio no (API).
- Iniciales sin nombre: se mezclan la parte anterior y la posterior a la `@` del email (`probe@test.com` → "PT"), en lugar de seguir la regla de "dos primeras letras" que se usa con un nombre de una sola palabra (API).
- Botón "Cerrar sesión": la pantalla tiene preparado el texto "Cerrando sesión…", pero nunca se llega a ver porque la sesión se cierra y la persona sale del perfil antes de que se pinte (pantalla de perfil).
- Mensaje de 401: el frontend muestra "Tu sesión ha caducado" ante cualquier 401, aunque los tokens no caducan; solo dejan de valer al cerrar sesión con ellos (frontend vs. API).
- Fecha de alta: la respuesta del registro lleva milisegundos (`.422`) y la misma fecha en el perfil vuelve truncada (`.000`) (API, registro vs. perfil).

### 3. Comportamientos que no supimos clasificar como bug o contrato

- **Email sensible a mayúsculas.** `ana@x.com` y `Ana@x.com` son cuentas distintas: se pueden registrar las dos, y desde una no se puede entrar escribiendo el email de la otra. *Contrato:* el email es un identificador exacto, y registro y login lo tratan igual, así que es coherente. *Bug:* para una persona es el mismo email, y permite cuentas duplicadas por un despiste al escribir. El código no normaliza el email en ningún punto, y no hay test ni comentario que indique la intención.
- **Iniciales a partir del email.** Sin nombre, `probe@test.com` da "PT". *Contrato:* es una regla deliberada para que siempre haya dos letras. *Bug:* la intención era usar solo la parte anterior a la `@` ("PR"), y el dominio se cuela porque se reutiliza la misma división que para nombre y apellido. El código no lo explica.
- **Servidor caído al recargar.** El frontend conserva el token pero muestra el login como si no hubiera sesión. *Contrato:* es deliberado; un comentario en el código lo justifica (no perder la sesión por un corte pasajero). *Bug:* la persona ve el login, vuelve a entrar y se crea un segundo token, sin que el primero se invalide nunca. El comentario explica por qué se conserva el token, pero no si mostrar el login en ese caso es lo deseado.
- **Tokens sin caducidad y sesiones múltiples.** Cada login crea un token que vale indefinidamente, y cerrar sesión solo invalida el de ese dispositivo. *Contrato:* una sesión "para siempre" por dispositivo es una decisión razonable para un MVP. *Bug:* es un olvido de configuración y debería haber caducidad o un "cerrar todas las sesiones". Desde fuera no se puede distinguir sin esperar mucho tiempo, y el código no dice nada.
