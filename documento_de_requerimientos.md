# Documento de requerimientos: app de acceso y seguridad residencial

**Contexto, actores, requerimientos funcionales y no funcionales, plataformas, integración con la pluma y decisiones pendientes.**

## 1. Contexto y objetivo

En muchos fraccionamientos de México el control de visitas se hace con libreta, llamadas o WhatsApp entre la caseta y los residentes. Esto es lento, deja poco registro y no ayuda en una emergencia. El producto es una plataforma de acceso y seguridad, del tipo de Kigo, con apps nativas para residentes y guardias (Swift en iPhone y Apple Watch; Kotlin en Android, Wear OS y la tablet de la caseta) y un panel web para la administración, que permita:

*   Autorizar visitas en segundos desde el teléfono, el reloj o con la voz.
*   Levantar la pluma de la caseta desde el sistema.
*   Pedir ayuda con un botón de pánico que envía la ubicación a los guardias.
*   Dar acceso a visitas recurrentes con códigos que tienen horario y vigencia.

| Concepto | Descripción |
| :--- | :--- |
| **Mercado** | Fraccionamientos y condominios horizontales en México |
| **Cliente que paga** | Administración o mesa directiva (cuota mensual por vivienda) |
| **Usuarios** | Residentes, guardias, administradores y visitas |
| **Estado** | Versión 0.6: agrega el módulo Centinela (rondines con vehículo semi-autónomo o teledirigido, de la plataforma) con servicios de vigilancia a pedido (sección 15). Incorpora las decisiones anteriores (secciones 12 a 14). Pendiente elegir el fraccionamiento piloto |
| **Pluma** | La provee y mantiene el fraccionamiento; el sistema solo agrega su controlador en la caseta |

## 2. Actores

| Actor | Qué hace | Dispositivos |
| :--- | :--- | :--- |
| **Residente** | Autoriza visitas, entra con el tag de su vehículo o con su Mi QR, administra recurrentes y activa el pánico | iPhone, Apple Watch, Android, Wear OS; otros relojes solo con avisos |
| **Guardia** | Inicia turno con su PIN personal, registra visitas, levanta la pluma, valida códigos, atiende alertas de pánico y opera al Centinela (rondines, revisiones y control manual) | App nativa en la tablet Android de la caseta (modo kiosco); opcionalmente teléfono y reloj |
| **Propietario no residente** | Dueño que renta la vivienda o la tiene deshabitada; entra al fraccionamiento con su QR y solicita permisos de obra. Si la vivienda está rentada, no autoriza visitas ni ve su bitácora | iPhone o Android |
| **Trabajador de obra** | Entra con un permiso de obra vigente, dentro del horario; el guardia lo registra por nombre | Ninguno |
| **Administrador** | Da de alta viviendas, residentes y guardias, configura reglas y consulta la bitácora | Panel web |
| **Emergencias** | Ambulancia, bomberos y policía; entran de inmediato con registro posterior | Ninguno |
| **Visita** | Llega a la caseta; el guardia la registra o escanea su código, y siempre se avisa al residente | Ninguno; no necesita la app. Si tiene invitación, un enlace con QR/PIN recibido por WhatsApp o SMS |
| **Servicio** | Paquetería, gas, agua, mantenimiento. Indica a qué vivienda va y sigue el mismo flujo que una visita; solo pasa con apertura manual del guardia si va con la administración o tiene permiso de ella | Ninguno, o un enlace con QR/PIN si tiene invitación |
| **Controlador de pluma** | Recibe órdenes del sistema y acciona la pluma que ya tiene el fraccionamiento; si el operador lo permite, lee sus señales de estado | Equipo con relevadores en la caseta, provisto por nosotros |
| **Tag de vehículo** | Pegatina en el parabrisas que identifica el vehículo del residente ante el lector del carril y abre la pluma sin usar el teléfono | Ninguno; lo provee el sistema |
| **Centinela** | Vehículo semi-autónomo o teledirigido de la plataforma que hace los rondines y los servicios de vigilancia a pedido; lo opera el guardia desde la tablet | Ninguno para el residente; se pide desde la app |

## 3. Requerimientos funcionales

**Prioridad:** Debe (indispensable para el MVP o su fase), Debería (importante, puede moverse de fase) y Podría (opcional según la demanda). La fase corresponde al plan de la sección 8. Los requerimientos nuevos se numeran desde RF-70 para no cambiar los existentes; entre paréntesis, la observación de la revisión que atienden (sección 12).

### 3.1 Visitas y autorización

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-01** | El guardia registra cada acceso con su tipo (Visita o Servicio), nombre, vivienda destino y, si quiere, placas y foto. En los servicios también captura la empresa. | Debe | 1 |
| **RF-02** | El residente recibe una notificación que indica el tipo (Visita o Servicio), con los botones "Autorizar" y "Rechazar". Rechazar funciona con el teléfono bloqueado; Autorizar pide desbloquearlo (Face ID, huella o código). Autorizar desde la notificación o por WhatsApp funciona desde cualquier lugar; no requiere estar cerca del fraccionamiento. (G-05, P-03) | Debe | 1 |
| **RF-03** | La notificación también aparece en Apple Watch y Wear OS con los mismos botones. | Debe | 1 |
| **RF-04** | Si el residente no responde en un tiempo configurable (unos 60 s), se escala en orden: notificación, WhatsApp y llamada. Al agotarse, el acceso queda "sin respuesta" y la visita no entra; nunca se autoriza sola. El guardia solo puede dejarla pasar como excepción, con motivo y permiso de la administración (RF-22). (1-02) | Debe | 1 |
| **RF-05** | El guardia ve la respuesta del residente al instante en la tablet. | Debe | 1 |
| **RF-06** | Si una vivienda tiene varios residentes, vale la primera respuesta (autorizar o rechazar) y se avisa a los demás quién respondió. (1-03) | Debe | 1 |
| **RF-07** | El residente crea una invitación única de tipo Visita o Servicio (nombre y fecha) y la comparte por WhatsApp o SMS. La visita no necesita la app: recibe un enlace a una página web con QR dinámico y un PIN de respaldo válido solo ese día y con usos limitados; cada uso queda registrado y se avisa al residente. (1-07) | Debe | 2 |
| **RF-08** | Toda visita o servicio se notifica al residente de la vivienda destino. Nadie entra solo con un QR o PIN: el código únicamente identifica a qué vivienda va y a quién avisar. Sin invitación, el residente debe autorizar (RF-02); con invitación o si es recurrente, el guardia valida el código y al residente solo se le avisa que llegó. | Debe | 1 |
| **RF-09** | Los servicios (paquetería, gas, agua, mantenimiento) deben indicar a qué vivienda van y siguen el mismo flujo que las visitas. Solo entran con la apertura manual del guardia, sin aviso a un residente, si van con la administración o tienen un permiso de ella; queda registrado el motivo y quién dio el permiso. | Debe | 1 |
| **RF-70** | Cada acceso pasa por estas etapas: registrado, esperando respuesta, autorizado, rechazado, sin respuesta, abrió, pasó, salió, cancelado y expirado. Solo el residente autoriza o rechaza; el guardia o el sistema marcan abrió, pasó y salió, y cada cambio registra quién lo hizo. "Pasó" solo se confirma si el operador da la señal (RF-79); si no, queda como no confirmado. (F-05) | Debe | 1 |
| **RF-71** | Un servicio puede llevar varias viviendas destino en un mismo acceso (pipa de gas, paquetería). Cada vivienda recibe su aviso y responde por separado, y el guardia ve cuáles autorizaron. (F-07) | Debería | 1 |
| **RF-72** | Cada vivienda elige cómo atender la paquetería: preguntar siempre, dejar en caseta o autorizar siempre. En todos los casos se avisa al residente. (F-08) | Debería | 1 |
| **RF-73** | Bitácora de paquetes recibidos en caseta: el guardia los registra, se avisa al residente y queda constancia de quién lo recogió y cuándo. (F-08) | Debería | 2 |
| **RF-74** | Se registra la salida de cada visita y servicio. Los accesos sin salida se cierran solos a una hora configurable, con una marca; el residente puede indicar que su visita se queda a dormir. (F-09) | Debería | 1 |
| **RF-75** | Invitación de evento con cupo y horario: un solo enlace para varios invitados, y cada uno se registra con su nombre al abrirlo o en la caseta. El residente recibe avisos agrupados, no uno por invitado. (F-10) | Debería | 2 |
| **RF-76** | Ambulancia, bomberos y policía son una categoría propia: el guardia abre de inmediato y captura los datos después. Se avisa a la administración y, si van a una vivienda, al residente. (F-06) | Debe | 1 |
| **RF-77** | La administración mantiene una lista de personas o placas con acceso restringido. Solo un administrador con ese permiso agrega o quita registros; un residente puede solicitarlo. Cada registro lleva quién lo agregó, el sustento (por ejemplo, una denuncia ante el Ministerio Público), una vigencia (propuesta: 90 días, renovable con sustento) y un segundo dato para coincidir (placa o identificación), no solo el nombre. Al vencer, el registro sale de la lista. (P-01) | Debería | 2 |
| **RF-86** | Si coinciden la placa o la identificación con la lista restringida, el acceso se detiene aunque la persona traiga un código vigente de otra vivienda o la autorización de un residente, y pasa a la administración como excepción (RF-22). La administración decide; si no responde, la persona no entra. El residente que la invitó o autorizó recibe aviso. Si solo coincide el nombre, la tablet muestra "posible coincidencia" y el guardia revisa la identificación antes de detener el acceso. (P-01) | Debería | 2 |

### 3.2 Visitas recurrentes y códigos de acceso

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-10** | El residente mantiene una lista de accesos recurrentes de tipo Visita (por ejemplo, un familiar) o Servicio (por ejemplo, la persona de limpieza), con días, horario y fecha de vencimiento. | Debe | 2 |
| **RF-11** | Cada visita recurrente tiene un código (QR dinámico que cambia cada 30 s, o PIN con usos limitados por día) que la identifica junto con la vivienda que la autorizó; se comparte por mensaje como enlace a una página web, sin que la visita instale la app. Cada uso queda registrado y se avisa al residente. (1-07) | Debe | 2 |
| **RF-12** | El residente puede revocar un código y el cambio aplica de inmediato en las casetas con conexión. Sin conexión, la tablet advierte al guardia cuando sus datos tienen más de 15 minutos; al reconectarse marca los accesos hechos con códigos ya revocados y avisa al residente. (C-02) | Debe | 2 |
| **RF-13** | El guardia escanea el código con la tablet, que muestra a qué vivienda va y quién la autorizó, y valida horario y vigencia aunque no haya internet en la caseta. | Debe | 2 |
| **RF-14** | Cada vez que entra una visita recurrente se avisa al residente. | Debe | 2 |
| **RF-15** | El residente tiene su propio QR de acceso, visible en el teléfono y en el reloj, para entrar a pie, en taxi o cuando el tag no lee; el guardia lo escanea y abre sin pedir autorización. Es el respaldo de los tags (RF-94). (1-08, T-01) | Debe | 2 |
| **RF-16** | Un lector QR/PIN en el carril peatonal o de visitas identifica la visita o al residente y avisa; la pluma solo se levanta cuando el residente autoriza o el guardia confirma. | Podría | 5 |

### 3.3 Apertura de la pluma

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-20** | El sistema levanta la pluma que ya tiene el fraccionamiento mediante un controlador en la caseta, con un lector de tags por carril vehicular y conectado a la entrada de solo apertura del operador, sin reemplazarlo ni modificarlo. (G-02) | Debe | 0 |
| **RF-21** | El guardia levanta la pluma desde la tablet con un botón por carril (entrada y salida) una vez que la visita está autorizada, cuando el tag no lee o para la salida. La tablet muestra la fila de vehículos con el estado de cada uno. (G-01) | Debe | 1 |
| **RF-22** | Toda apertura manual del guardia requiere un motivo de un catálogo fijo (servicio de la administración, servicio con permiso, emergencia, excepción y otros). Las excepciones quedan marcadas y piden el permiso de la administración. (1-06, F-06) | Debe | 1 |
| **RF-23** | El residente no tiene botón de apertura en la app, el reloj ni los widgets: su vehículo entra con el tag del parabrisas (RF-93 a RF-96) y cualquier persona entra a pie o en otro vehículo con su Mi QR (RF-15). Las visitas y los servicios siguen el flujo de autorización (RF-02). (T-01) | Debe | 1 |
| **RF-24** | El administrador puede levantar la pluma desde el panel web, con doble confirmación. | Debería | 2 |
| **RF-25** | Si no hay internet, la tablet del guardia sigue levantando la pluma por la red local. Cada tablet tiene su propia llave, registrada y revocable, con la que firma las órdenes; el controlador guarda esas aperturas y las reporta al recuperar la conexión. (C-01) | Debe | 1 |
| **RF-26** | Se descarta la apertura por Bluetooth del controlador: el acceso del residente es por tag (RF-93) o por Mi QR (RF-15). El Bluetooth deja de ser una mejora planeada. (T-01) | Podría | 5 |
| **RF-27** | El botón físico y el control remoto de la pluma siguen funcionando como respaldo y no pasan por el controlador. Si el operador tiene una señal de posición o de estado, el controlador detecta esas aperturas y las registra; si no la tiene, el guardia indica en la tablet la apertura manual, con su motivo (RF-22). (G-03, R-06) | Debe | 0 |
| **RF-28** | Cada apertura registra quién (guardia por su PIN, o tag con su vehículo), cuándo, desde qué medio, en qué carril y por qué motivo. (F-02, T-01) | Debe | 0 |
| **RF-78** | Cada guardia inicia turno en la tablet con su PIN personal, también sin internet; la bitácora registra qué guardia hizo cada acción. (F-02) | Debe | 1 |
| **RF-79** | Si el operador de la pluma tiene salidas disponibles (detector de vehículo, final de carrera, luz de estado), el controlador las lee sin modificar la pluma y reporta vehículo detectado, vehículo que pasó, apertura con botón físico o control y posición de la pluma. Donde no las haya, el sistema opera solo con el pulso. (F-04, G-03) | Debería | 1 |
| **RF-80** | Tiempo mínimo entre dos pulsos al mismo carril (unos 5 s, configurable): las órdenes repetidas en ese lapso se ignoran, para que dos pulsos seguidos no detengan ni bajen la pluma. Límite de aperturas por usuario (por ejemplo, 10 por hora, configurable). (1-06, G-02) | Debe | 0 |

### 3.4 Control por voz

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-30** | En iPhone y Apple Watch, el residente autoriza la visita pendiente con Siri ("Oye Siri, autoriza la visita en [nombre de la app]"). | Debe | 4 |
| **RF-31** | No hay comando de voz para abrir la pluma: la apertura del residente es por tag (RF-93). Siri se usa para autorizar visitas (RF-30), activar el pánico y consultar si hay visitas esperando. (T-01) | Debe | 4 |
| **RF-32** | Los comandos de voz solo funcionan con el dispositivo desbloqueado. | Debe | 4 |
| **RF-33** | Si hay más de una visita pendiente, la app pregunta cuál autorizar. | Debe | 4 |
| **RF-34** | Las acciones de autorizar visita pendiente y de pánico se pueden asignar al botón de Acción del iPhone y del Apple Watch Ultra. | Debería | 4 |
| **RF-35** | En Android se ofrecen atajos y acceso rápido (Quick Settings) para visitas y pánico. No se incluye control por voz, porque Google sustituyó Assistant por Gemini en los teléfonos. (G-09) | Debería | 4 |

### 3.5 Botón de pánico

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-40** | El residente activa el pánico manteniendo presionado unos 3 s; tiene unos segundos para cancelarlo y evitar falsas alarmas. | Debe | 3 |
| **RF-41** | Se puede activar desde la app, desde un widget, desde la pantalla bloqueada y desde el reloj (ícono en la carátula en Apple Watch; Tile en Wear OS). En widgets y controles, donde no se puede mantener presionado, un toque abre la cuenta regresiva cancelable. (G-06) | Debe | 3 |
| **RF-42** | Dentro de la geocerca del fraccionamiento, la alerta envía la vivienda, los datos del residente y su ubicación en vivo a todos los guardias en turno. Fuera del fraccionamiento, va a los contactos de emergencia y se ofrece marcar al 911. (1-09) | Debe | 3 |
| **RF-43** | La alerta suena en los dispositivos del guardia aunque estén en silencio. En la tablet (modo kiosco) la app está siempre al frente y el permiso de pantalla completa se activa al instalarla; en teléfonos con Android 14 o posterior se le pide al usuario. Critical Alerts de iOS solo si algún guardia usa iPhone. (G-07, G-08) | Debe | 3 |
| **RF-44** | El guardia confirma que atiende la alerta; si nadie confirma en un tiempo configurable, se escala a los demás contactos. La empresa de seguridad define por escrito el protocolo de atención (por ejemplo, si el guardia puede dejar la caseta). (1-09) | Debe | 3 |
| **RF-45** | Se avisa a los contactos de emergencia del residente y se ofrece marcar al 911 desde el mismo flujo. | Debería | 3 |
| **RF-46** | Detección de caídas en el reloj que dispara una alerta, según la geocerca del fraccionamiento: dentro, avisa a los guardias en turno con la vivienda, los datos del residente y su ubicación en vivo, con el mismo flujo del pánico (confirmación y escalamiento, RF-42 y RF-44); fuera, solo avisa y manda la ubicación a los integrantes de la vivienda. | Podría | 5 |

### 3.6 Administración y bitácora

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-50** | El administrador da de alta y de baja viviendas, residentes y guardias, e invita a los residentes por enlace. | Debe | 1 |
| **RF-51** | Bitácora consultable y exportable de visitas, aperturas y alertas, con filtros por fecha, vivienda y tipo. | Debe | 1 |
| **RF-52** | Configuración del fraccionamiento: casetas y, en cada una, sus carriles; cada carril es vehicular o peatonal, de entrada o de salida, y los vehiculares llevan su lector de tags. También el perímetro del fraccionamiento (geocerca del fraccionamiento), duración del pulso, tiempo mínimo entre pulsos y tiempos de escalamiento. (F-01, T-01) | Debe | 1 |
| **RF-53** | Un administrador puede gestionar varios fraccionamientos desde la misma cuenta. | Debería | 2 |
| **RF-54** | Monitoreo del estado del controlador y de los lectores de tags (en línea, última lectura, última apertura y, si el operador lo permite, posición de la pluma) con aviso si se desconectan. | Debe | 1 |
| **RF-55** | La administración registra permisos para servicios del fraccionamiento (jardinería, mantenimiento, proveedores de la administración) con vigencia y horario; el guardia los ve en la tablet. | Debería | 1 |
| **RF-81** | La vivienda tiene un campo de estado de pago, apagado por defecto y sujeto a revisión legal. Aunque se active, nunca restringe la entrada del residente a su vivienda, sus visitas ni el pánico. (F-03) | Podría | - |
| **RF-82** | Bitácora de las acciones de los administradores: altas, bajas, aprobaciones, permisos y cambios de configuración. | Debe | 1 |
| **RF-83** | Las viviendas cuyos residentes no usan la app reciben el aviso y autorizan por WhatsApp (botones de la plantilla) o por llamada. (G-04) | Debe | 1 |
| **RF-88** | Permiso de obra por periodo, ligado a la vivienda o al lote: lo solicita el propietario y lo aprueba la administración. Lleva responsable (maestro de obra o Director Responsable de Obra), número de licencia de construcción, fechas y el horario de obra del reglamento. Dentro de ese horario el guardia registra a cada trabajador por nombre al entrar, sin pedir autorización diaria al dueño; fuera del horario aplica el flujo normal (RF-04). La lista restringida (RF-77) también aplica a los trabajadores. (P-03) | Debería | 2 |
| **RF-89** | El dueño recibe un resumen diario de las entradas a su obra y un aviso inmediato si alguien entra fuera del horario de obra o coincide con la lista restringida. (P-03) | Debería | 2 |
| **RF-90** | Vivienda con comercio: la administración puede marcar una vivienda como comercio, con horario de atención, solo después de subir al panel el uso de suelo compatible o la licencia de funcionamiento. Dentro de ese horario el guardia registra a cada cliente y el residente recibe un aviso agrupado, sin autorizar uno por uno; fuera del horario los clientes siguen el flujo normal de visita. El aviso agrupado va como notificación de la app, y por WhatsApp solo a quien no la tiene. (P-04) | Debería | 2 |
| **RF-91** | Si una vivienda sin comercio registrado recibe un volumen atípico de visitas, el panel lo informa a la administración. El sistema no sanciona: eso le corresponde al municipio, y a la asociación conforme a su reglamento. (P-04) | Podría | 2 |
| **RF-92** | Huésped temporal (renta vacacional): el titular o el propietario lo da de alta con fechas de entrada y salida. Durante su estancia entra con su propio QR y el titular recibe aviso de cada entrada; al terminar, el acceso vence solo. No convierte la vivienda en comercio. (P-04) | Podría | 2 |
| **RF-93** | Cada vehículo de residente lleva un tag adherido al parabrisas (pegatina), ligado a la vivienda, al modelo y color del auto y a la placa. Los da de alta la administración en el panel, donde también se bloquean, se sustituyen por extravío o robo, y se revocan. La administración fija el máximo de tags por vivienda. (T-01) | Debe | 2 |
| **RF-94** | Cada carril vehicular tiene un lector de tags conectado al controlador. Al leer un tag vigente, el controlador manda el pulso de apertura (RF-20) y registra la entrada con el tag, la placa y el carril. Funciona también sin internet, porque el controlador guarda la lista de tags vigentes y la sincroniza con la nube; en la lectura por carril no aplica la distinción de exclusivo o compartido. (T-01) | Debe | 2 |
| **RF-95** | Lecturas repetidas del mismo tag en el mismo carril dentro del tiempo mínimo entre pulsos (RF-80) generan un solo pulso y un solo registro. La vivienda recibe un aviso de entrada de su vehículo (RF-08). Un tag bloqueado, revocado o desconocido no abre: en carril exclusivo el guardia indica en la tablet que abra manual o que el residente muestre su Mi QR (RF-15); en compartido sigue el flujo de visita. (T-01) | Debe | 2 |
| **RF-96** | Anti-clonado de tags: la lectura se comprueba con el vehículo registrado (placa o foto del acceso, cuando haya cámara) y con el historial de entradas; un tag leído en dos lugares sin tiempo plausible de traslado marca una coincidencia sospechosa y la administración decide. Si el piloto requiere más resistencia, se evalúan tags con clave cifrada. (T-01) | Podría | 2 |
| **RF-97** | La app muestra los vehículos y tags de la vivienda (TAG, auto, estado: activo, bloqueado o revocado) y el aviso de cada entrada. Desde ahí se reporta un tag extraviado (la solicitud va a la administración, que lo bloquea); no se puede dar de alta un tag desde la app. (T-01) | Debería | 2 |

### 3.7 Registro y alta de residentes

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-60** | La administración registra el fraccionamiento (por ejemplo, Bosques Sanctorum) y sus viviendas en el panel web; a partir de ahí aparece en la app para que los residentes lo encuentren. | Debe | 1 |
| **RF-61** | El residente descarga la app, busca su fraccionamiento o entra con un enlace o código de invitación, y solicita su alta indicando su vivienda y su teléfono verificado por SMS. | Debe | 1 |
| **RF-62** | La administración aprueba o rechaza cada solicitud. Mientras no esté aprobada, la app no muestra el Mi QR, los vehículos ni permite autorizar visitas. | Debe | 1 |
| **RF-63** | Si la administración precargó al residente con su teléfono, el alta se aprueba automáticamente al verificar el número. | Debería | 1 |
| **RF-64** | El titular de la vivienda invita a los integrantes de su familia sin pasar por la administración; la administración puede ver y revocar esos accesos. Cualquier integrante adulto autoriza visitas; solo el titular agrega o quita integrantes. (1-04) | Debe | 1 |
| **RF-65** | Al dar de baja a un residente (por ejemplo, porque se mudó), pierde el acceso de inmediato en todos sus dispositivos. | Debe | 1 |
| **RF-66** | Cada cuenta admite hasta 3 dispositivos. Registrar uno nuevo genera aviso al titular y a la propia cuenta. (1-05) | Debe | 1 |
| **RF-67** | Las acciones sensibles piden biometría del teléfono (Face ID o Touch ID en iOS; huella o rostro con BiometricPrompt en Android), con el código del teléfono como respaldo: crear o editar recurrentes y autorizaciones automáticas, cambiar datos de la cuenta (teléfono, dispositivos, integrantes de la familia, contactos de emergencia) y reportar o administrar tags. Autorizar desde la notificación solo requiere el teléfono desbloqueado y una invitación única no pide biometría. | Debe | 1 |
| **RF-68** | En el reloj se muestran las notificaciones con Autorizar/Rechazar, el Mi QR y el pánico; crear recurrentes y cambiar la cuenta se hacen desde el teléfono. | Debe | 2 |
| **RF-84** | Cada vivienda indica si el titular es propietario o arrendatario. Para arrendatarios, la administración puede fijar la fecha de fin del contrato; al llegar, pide confirmar o da de baja el acceso. Si el titular es el arrendatario, el propietario puede tener acceso como propietario no residente (RF-87). | Debería | 2 |
| **RF-87** | Propietario no residente: la administración da de alta al dueño que no vive en la vivienda (porque la renta o está deshabitada) después de comprobar la propiedad. Tiene su propio QR (como RF-15) para entrar al fraccionamiento a cualquier hora, pero no usa el botón de apertura. Si la vivienda está rentada, el arrendatario recibe aviso de cada entrada del propietario, y el propietario no autoriza visitas ni ve la bitácora de la vivienda; si lleva a alguien a la casa, esa persona entra como visita o servicio de la vivienda y la autoriza el arrendatario. Si está deshabitada, el propietario recibe los avisos de la vivienda y autoriza desde cualquier lugar (RF-02). Entrar al fraccionamiento no es entrar a la casa. (P-02, P-03) | Debería | 2 |

### 3.8 Centinela y rondines

| ID | Requerimiento | Prioridad | Fase |
| :--- | :--- | :--- | :--- |
| **RF-100** | Revisión de vivienda a pedido: el residente solicita desde la app que el Centinela pase frente a su domicilio y recibe fotografías o un video breve más la confirmación del estado visible de la puerta, cochera, ventanas, vehículo y exterior, dictaminada por el guardia que lo opera. (L-01) | Debe | 6 |
| **RF-101** | Revisión inmediata: el residente reporta que vio a una persona extraña frente a su casa, escuchó un ruido en el exterior o hay un vehículo detenido desde hace tiempo. Va primero el Centinela a verificar a distancia; si el guardia confirma un riesgo, el reporte escala como alerta a los guardias en turno y a la administración. (L-03) | Debe | 6 |
| **RF-102** | Modo Vacaciones: el residente indica los días en que estará fuera y el sistema programa revisiones aleatorias de su domicilio, con un informe diario o cada cierto número de días. Registra a las personas autorizadas para ingresar durante su ausencia: esas personas entran con su QR sin pedir autorización extra (RF-15). (L-02) | Debería | 6 |
| **RF-103** | El residente consulta en la app cuándo pasó el Centinela por su calle: hora, zona recorrida y estatus del rondín (hecho, con novedad, interrumpido). (P-01) | Debería | 6 |
| **RF-104** | Llegada Segura: el residente pide que el Centinela lo espere en el acceso y acompañe su vehículo hasta su casa; pensado para la madrugada o horarios de poca actividad. Se avisa al confirmar y al terminar el acompañamiento. (P-02) | Debería | 6 |
| **RF-105** | Vigilancia temporal: el residente pide que el Centinela permanezca un periodo frente a su domicilio, por portón descompuesto, mudanza, fiesta o reunión, incidente reciente, vehículo sospechoso o una reparación con trabajador; funciona como presencia preventiva y punto de observación. (P-05) | Debería | 6 |
| **RF-106** | Avisos del rondín y de las revisiones: si el Centinela detecta una cochera o portón que parece abierto, especialmente de noche, o un paquete visible en la entrada, el residente recibe una notificación para confirmar la situación; no se considera emergencia sin su confirmación. (P-04, L-02) | Debería | 6 |
| **RF-107** | Contacto directo con seguridad desde la app: chat, llamada o intercomunicador con el guardia en turno, sin depender de llamar a la caseta. (P-03) | Debería | 6 |
| **RF-108** | El guardia de la caseta opera al Centinela desde la tablet: asigna y pausa rondines, toma los pedidos del residente, revisa fotos y video y dictamina el estado visible, y toma control manual si hace falta. (C-04) | Debe | 6 |
| **RF-109** | La administración programa los rondines por zonas y horarios; cada rondín queda en la bitácora con hora, zona y resultado. El rondín físico del guardia deja de ser el mecanismo principal de vigilancia. | Debe | 6 |
| **RF-110** | Los pedidos se atienden por prioridad: primero las alertas del pánico y las revisiones inmediatas, después Llegada Segura, revisiones a pedido, rondines programados y vigilancia temporal. La app muestra el turno estimado de cada pedido. | Debería | 6 |
| **RF-111** | Las fotos y el video del Centinela van cifrados, son visibles solo para el residente que los pidió, el guardia que los revisó y la administración, y se borran al vencer su plazo (RNF-06). (L-01) | Debe | 6 |
| **RF-112** | El residente ve el estado del Centinela (disponible, en rondín, en revisión, cargando, fuera de servicio) y el tiempo estimado de llegada cuando pide un servicio. | Debería | 6 |

## 4. Plataformas

| Plataforma | Nivel | Notas |
| :--- | :--- | :--- |
| **iPhone** | App nativa completa | Swift/SwiftUI |
| **Apple Watch** | App nativa | Avisos con Autorizar/Rechazar, Mi QR y pánico; funciona sin el iPhone en modelos con datos móviles |
| **Android** | App nativa completa | Kotlin / Jetpack Compose; voz limitada |
| **Wear OS** (Galaxy Watch 4 en adelante, Pixel Watch y otros) | App nativa | Mismas funciones que en Apple Watch |
| **Xiaomi, Huawei, Amazfit, Samsung con Tizen** | Solo avisos | Reciben la notificación que refleja el teléfono, normalmente sin botones; mini apps en Huawei o Amazfit como opción futura |
| **Tablet Android (guardia)** | App nativa en Kotlin, en modo kiosco | Funciona sin internet; abre la pluma por red local y opera al Centinela |
| **Centinela** | Vehículo semi-autónomo o teledirigido con cámaras, de la plataforma | Lo opera el guardia desde la tablet; se coordina por la app |
| **Web (administración)** | Panel web | Navegadores actuales |
| **CarPlay / Android Auto** | Opcional | Sujeto a las categorías de app que permiten Apple y Google; con Siri funciona desde CarPlay sin una app dedicada |

## 5. Requerimientos no funcionales

| ID | Categoría | Requerimiento |
| :--- | :--- | :--- |
| **RNF-01** | Rendimiento | La notificación llega al residente en menos de 5 s y la pluma se levanta en menos de 2 s desde la orden del guardia o la lectura del tag, con buena conexión. (T-01) |
| **RNF-02** | Disponibilidad | 99.9% mensual del servicio en la nube. La caseta sigue operando sin internet: aperturas por guardia y validación de códigos. Todo lo que el guardia registra sin conexión (visitas, aperturas manuales, trabajadores de obra, clientes y excepciones) se guarda en la tablet y se envía al recuperar la conexión. La tablet guarda, cifrados, solo los códigos vigentes, los permisos de servicios y de obra, y la lista restringida con nombre, placa e identificación, sin el sustento; no el directorio de teléfonos de los residentes. (F-11, P-01, R-07) |
| **RNF-03** | Seguridad | Las órdenes de apertura de la tablet van firmadas, expiran en segundos y no se pueden reutilizar. Cada controlador tiene su propio certificado (TLS mutuo) y conexión saliente, sin puertos abiertos hacia internet. En la red local solo acepta órdenes firmadas por tablets registradas (RF-25) y las lecturas válidas de sus lectores de tags (RF-94). Los límites de aperturas están en RF-80. (C-01, 1-06, T-01) |
| **RNF-04** | Seguridad física | El sistema solo manda el pulso de apertura; nunca controla el cierre ni anula los sensores de seguridad de la pluma. |
| **RNF-05** | Autenticación | Inicio de sesión con teléfono y código y sesiones por dispositivo. La biometría de RF-67 está atada a la llave del dispositivo (Secure Enclave en iOS, Android Keystore con BIOMETRIC_STRONG): el servidor solo acepta esas acciones firmadas después de la biometría, así que no basta con alterar la app. Los administradores usan doble factor. |
| **RNF-06** | Privacidad | Cumplimiento de la Ley Federal de Protección de Datos Personales en Posesión de los Particulares vigente desde el 21 de marzo de 2025. La administración es la responsable de los datos y la plataforma es encargada, con contrato de tratamiento. Aviso de privacidad visible en la caseta para las visitas, redactado con la ley vigente y revisado por un abogado. Fotos, placas y ubicación se borran tras un plazo que fija la administración. Cifrado en tránsito y en reposo. (G-11) |
| **RNF-07** | Auditoría | Los eventos de la bitácora son permanentes: nadie los edita ni los borra. Los datos personales asociados (fotos, placas, ubicación) se eliminan al vencer su plazo (RNF-06) y el evento conserva una marca de que existieron. (C-03) |
| **RNF-08** | Usabilidad (guardia) | Botones grandes y pocos pasos, para guardias con poca experiencia tecnológica. La capacitación debe tomar menos de 30 minutos. |
| **RNF-09** | Idioma | Español (México) en el lanzamiento; preparado para agregar otros idiomas. |
| **RNF-10** | Accesibilidad | Compatible con VoiceOver, TalkBack y texto grande. |
| **RNF-11** | Escalabilidad | Múltiples fraccionamientos en una sola base PostgreSQL, con el identificador del fraccionamiento en cada tabla y Row-Level Security que impide leer filas de otro fraccionamiento, además del filtro en el backend. (T-01, T-02) |
| **RNF-12** | Hardware en caseta | El controlador tiene respaldo de energía (UPS) y puede usar 4G como respaldo de red. |
| **RNF-13** | Compatibilidad con la pluma | La pluma la provee y la mantiene el fraccionamiento; el sistema no la garantiza ni le da mantenimiento. Funciona con cualquier operador que tenga entrada de contacto seco o de pulsador. Si hay que ajustar la tarjeta (por ejemplo, al modo de solo apertura), lo hace el proveedor de la pluma. |

## 6. Integración con la pluma

La pluma es del fraccionamiento y no se reemplaza ni se modifica. Se agrega un controlador (ESP32, Raspberry Pi o un módulo industrial de relevadores) con un relevador por carril. El controlador se conecta a la entrada de solo apertura (contacto seco o pulsador) del operador de la pluma actual y mantiene una conexión saliente y cifrada (MQTT/TLS) con la nube. Cuando recibe una orden válida (de la tablet o del panel web) o lee un tag vigente, manda un pulso de alrededor de 1 s, confirma la ejecución y la registra en su bitácora local, aunque no haya internet. Si el operador tiene salidas de estado (detector de vehículo, final de carrera), el controlador también las lee (RF-79); si no, opera solo con el pulso.

Algunas tarjetas alternan abrir, parar y cerrar con cada pulso; por eso el controlador va a la entrada de solo apertura y respeta un tiempo mínimo entre pulsos (RF-80). El botón físico y el control remoto no pasan por el controlador, para que sigan funcionando si este falla (RF-27).

**Levantamiento por caseta (antes de instalar):**
Marca y modelo del operador de la pluma; si tiene entrada de solo apertura, de contacto seco o de pulsador; qué salidas de estado tiene (detector de vehículo, final de carrera, luz); cuántos carriles hay, si son vehiculares o peatonales, de entrada o salida, y exclusivos o compartidos; si hay espacio de espera y retorno para un vehículo rechazado; qué red hay (Ethernet, Wi-Fi o 4G); si hay contacto eléctrico; quién da mantenimiento a la pluma, y si ya usan tarjetas, tags o controles remotos que haya que conservar, y la distancia de lectura disponible para ubicar el lector de tags.

## 7. Supuestos, restricciones y dependencias externas

*   Apps nativas sin lógica compartida: Swift en iPhone y Apple Watch; Kotlin en Android, Wear OS y la tablet del guardia. Solo la administración usa un panel web.
*   Las notificaciones push no garantizan la entrega, por eso existe el respaldo de RF-04.
*   Apple aprueba caso por caso el permiso de Critical Alerts (RF-43) y el de detección de caídas (RF-46).
*   Para Siri, la frase debe incluir el nombre de la app.
*   Google sustituyó Assistant por Gemini en los teléfonos Android, por lo que no hay control por voz en Android (RF-35).
*   WhatsApp cobra por mensaje entregado; la revisión estima unos USD 0.0085 por plantilla de utilidad en México. Hay que confirmar la tarifa vigente y estimar con el volumen real de avisos.
*   La pluma la provee y la mantiene el fraccionamiento (RNF-13); los lectores y los tags son del sistema y la administración los administra (RF-93, RF-94).
*   El Centinela es de la plataforma: vehículo, software de autonomía o teledirección y operación. Lo opera el guardia de la caseta y sus servicios van incluidos en la cuota mensual, sin costo por pedido.
*   El backend se desarrolla en NestJS y TypeScript con PostgreSQL.
*   El producto no sustituye al 911 ni a un servicio de monitoreo profesional.
*   El fraccionamiento proporciona la red y la electricidad en la caseta, y la tablet del guardia (o se incluye en el paquete).
*   Las capacidades de cada plataforma deben confirmarse con la documentación actual de Apple y Google antes de comprometer fechas.

## 8. Fases

| Fase | Contenido | Criterio de salida |
| :--- | :--- | :--- |
| **0. Piloto de la pluma** | Controlador, lectores y unos tags de prueba instalados; backend mínimo | Levantamiento de la caseta completo y una semana de aperturas confiables por tag, remotas y locales |
| **1. MVP** | Visitas, notificación con botones (también en el reloj), respaldo por llamada o WhatsApp, apertura por guardia, bitácora y panel básico | Más del 90% de los accesos registrados en la tablet los resuelve el residente (app, WhatsApp o llamada), contando a los residentes sin app |
| **2. Recurrentes y relojes** | Visitas recurrentes, QR dinámico, QR del residente, acceso por tag de vehículo, apps de Apple Watch y Wear OS | Los códigos y los tags se validan sin internet y un tag bloqueado no abre |
| **3. Pánico** | Pánico en teléfono y relojes, alertas a guardias y escalamiento | Prueba completa con el teléfono en silencio y la app cerrada |
| **4. Voz** | Siri en iPhone y Apple Watch (autorizar, pánico y consultar visitas), botón de Acción, atajos en Android | Comandos solo con el dispositivo desbloqueado y confirmación si hay varias visitas pendientes |
| **5. Opcionales** | Lector QR en el carril, lectura de placas con cámara (anti-clonado de tags), detección de caídas, otros relojes, CarPlay / Android Auto | Según la demanda |
| **6. Centinela** | Rondines programados, revisión de vivienda a pedido, revisión inmediata, Modo Vacaciones, Llegada Segura, vigilancia temporal, avisos de portón y paquete, contacto con el operador desde la app y consola en la tablet | Una revisión a pedido de punta a punta (pedido, recorrido, fotos, dictamen del guardia) y un Modo Vacaciones programado funcionando |

## 9. Decisiones pendientes

| Decisión | Opciones o propuesta | Bloquea |
| :--- | :--- | :--- |
| **Fraccionamiento piloto y modelo de su pluma** | | Fase 0 |
| **Tags:** proveedor, tipo (pegatina UHF estándar o con clave), cuántos por vivienda y si los paga el residente o la administración | Propuesta: pegatina UHF/EPC estándar al inicio, con verificación por vehículo (RF-96) | Fase 0 |
| **Centinelas por fraccionamiento, tiempo de respuesta por servicio y retención del video de las revisiones** | Se define con la empresa de seguridad y la demanda del piloto | Fase 6 |
| **Valores finales:** escalamiento, usos del PIN por día, límite de aperturas, hora de cierre de accesos, retención de datos, vigencia de la lista restringida | Los propuestos en RF-04, RF-07, RF-74, RF-77, RF-80 y RNF-06; ajustar en el piloto | Fase 1 |
| **Revisión legal** | Aviso de privacidad con la ley de 2025, contrato de encargado, campo de morosos (RF-81), lista restringida (RF-77, RF-86), comercio en viviendas con la ley de fraccionamientos de Puebla (RF-90, RF-91) y tratamiento de datos del tag (placa del vehículo y entradas) (RF-93) | Fase 1 |
| **Dispositivos del guardia en rondines** | Además de la tablet fija: teléfono, reloj o ninguno | Fase 1 |
| **Costo del respaldo por WhatsApp o SMS** | Incluido en la cuota o como complemento; estimar con el volumen real de avisos y repartos | Fase 1 |
| **Nombre de la app** | Corto y fácil de pronunciar, porque va dentro de la frase de Siri | Fase 4 |

## 10. Fuera de alcance

CCTV y videoportero, cobro de cuotas de mantenimiento, reservas de áreas comunes, comunicados a residentes e integración con monitoreo profesional o con el C5. Pueden considerarse en versiones posteriores.

## 11. Glosario

*   **Pluma:** Barrera vehicular de la caseta de acceso.
*   **Contacto seco:** Entrada del operador de la pluma que la abre cuando se cierra un circuito, igual que un botón.
*   **Geocerca del fraccionamiento:** Perímetro de todo el fraccionamiento (por ejemplo, Bosques Sanctorum). Define si una alerta de pánico o de caída va a los guardias o solo a la familia o los contactos.
*   **Propietario no residente:** Dueño que no vive en la vivienda porque la renta o está deshabitada. Entra al fraccionamiento con su QR, pero no a la casa.
*   **Permiso de obra:** Autorización por periodo, aprobada por la administración, para que una cuadrilla entre a una obra en su horario sin autorización diaria del dueño.
*   **Vivienda con comercio:** Vivienda con uso de suelo o licencia que permite atender clientes en un horario; sus clientes entran con registro del guardia y aviso agrupado.
*   **Tag:** Pegatina en el parabrisas del vehículo del residente; el lector del carril la identifica y abre la pluma sin usar el teléfono.
*   **Mi QR:** Código del residente que cambia cada 30 s; el guardia lo escanea para entrar a pie, en taxi o cuando el tag no lee.
*   **Centinela:** Vehículo semi-autónomo o teledirigido de la plataforma que hace los rondines y la vigilancia a pedido; lo opera el guardia desde la tablet.
*   **Rondín:** Recorrido programado del Centinela por zonas y horarios; queda en la bitácora con hora, zona y resultado.
*   **Revisión Inmediata:** Pedido urgente para que el Centinela verifique a distancia algo que el residente reportó (persona extraña, ruido, vehículo detenido).
*   **Modo Vacaciones:** Periodo en que el residente está fuera; incluye revisiones aleatorias, informes periódicos y una lista de personas autorizadas a ingresar.
*   **Llegada Segura:** El Centinela espera al residente en el acceso y acompaña su vehículo hasta su casa.
*   **Vigilancia temporal:** El Centinela permanece un periodo frente al domicilio como presencia preventiva y punto de observación.
*   **Carril exclusivo / compartido:** Exclusivo: solo lo usan residentes; si el tag no lee, el guardia abre manual o valida el Mi QR. Compartido: lo usan también visitas y servicios; un vehículo sin tag sigue el flujo de visita.
*   **QR dinámico:** Código que cambia cada pocos segundos para que no sirva una captura de pantalla.
*   **Critical Alerts:** Permiso de iOS que hace sonar una notificación aunque el teléfono esté en silencio.
*   **Tile/icono en la carátula:** Accesos rápidos en el reloj, sin abrir la app.
*   **MQTT:** Protocolo ligero de mensajes entre la nube y el controlador.
*   **LFPDPPP:** Ley Federal de Protección de Datos Personales en Posesión de los Particulares; la vigente se publicó en 2025 y sustituyó a la de 2010.

## 12. Cambios de la versión 0.2

Cómo se atendió cada observación de la revisión del documento 0.1, más los puntos agregados.

| Obs. | Tema | Dónde se atiende |
| :--- | :--- | :--- |
| **F-01** | Accesos peatonales y varias casetas | RF-52 |
| **F-02** | Identidad del guardia | RF-28, RF-78 |
| **F-03** | Residentes morosos | RF-81 (con la regla de no restringir entrada, visitas ni pánico) |
| **F-04** | Información del controlador | RF-79, opcional según el operador |
| **F-05** | Etapas de un acceso | RF-70 |
| **F-06** | Emergencias | RF-76, RF-22 |
| **F-07** | Proveedores con varias viviendas | RF-71 |
| **F-08** | Preferencias de reparto | RF-72, RF-73 |
| **F-09** | Registro de salidas | RF-74 (con opción de quedarse a dormir) |
| **F-10** | Invitaciones para eventos | RF-75 (con avisos agrupados) |
| **F-11** | Datos en la tablet sin internet | RNF-02 |
| **C-01** | Aperturas sin internet | RF-25, RNF-03 (sin puertos abiertos hacia internet) |
| **C-02** | Revocación de códigos | RF-12 |
| **C-03** | Bitácora y retención | RNF-07 |
| **1-01** | Apertura por el residente | RF-23, RF-31, RF-26, RF-52: exclusivo abre directo; compartido, solicitud de paso |
| **1-02** | Residente que no responde | RF-04: sin respuesta la visita no entra; excepción con permiso |
| **1-03** | Varios residentes | RF-06 |
| **1-04** | Permisos en la vivienda | RF-64 |
| **1-05** | Dispositivos por cuenta | RF-66 |
| **1-06** | Límites y motivos | RF-22, RF-80 |
| **1-07** | QR dinámico y PIN | RF-07, RF-11 |
| **1-08** | QR del residente | RF-15 sube a Debe |
| **1-09** | Alcance del pánico | RF-42, RF-44 |
| **T-01** | Base de datos | RNF-11: PostgreSQL |
| **T-02** | Separación por fraccionamiento | RNF-11: Row-Level Security |
| **G-01** | Fila en la caseta | RF-21, levantamiento de la sección 6 |
| **G-02** | Lógica de la tarjeta | RF-20, RF-80, sección 6 |
| **G-03** | Botón físico y control | RF-27, RF-79: no pasan por el controlador |
| **G-04** | Residentes sin la app | RF-83, criterio de la fase 1 |
| **G-05** | Autorizar con pantalla bloqueada | RF-02: rechazar bloqueado, autorizar desbloqueado |
| **G-06** | Pánico desde un widget | RF-41 |
| **G-07** | Pantalla completa en Android | RF-43, también en la tablet |
| **G-08** | Critical Alerts | RF-43 |
| **G-09** | Voz en Android | RF-35, sección 7 |
| **G-10** | Costo de WhatsApp | Secciones 7 y 9 |
| **G-11** | Privacidad de la visita | RNF-06 |
| **Nuevo** | Lista de acceso restringido | RF-77 |
| **Nuevo** | Bitácora de administradores | RF-82 |
| **Nuevo** | Propietario o arrendatario | RF-84 |
| **Nuevo** | Paquetes en caseta | RF-73 |
| **Nuevo** | Contrato de encargado de datos | RNF-06 |
| **Nuevo** | La pluma es del fraccionamiento | RNF-13, sección 6 |
| **Nuevo** | Caídas según la geocerca del fraccionamiento | RF-46, RF-52 |
| **Decisión** | Apps nativas en Swift y Kotlin (también la tablet), sin lógica compartida; panel web solo para la administración | Secciones 1, 2, 4, 7 y 9 |

## 13. Cambios de la versión 0.4

Puntos a aclarar del panel administrativo y decisiones posteriores.

| Punto | Decisión | Dónde se atiende |
| :--- | :--- | :--- |
| **P-01** | Persona restringida con código vigente: el acceso se detiene y decide la administración; sin respuesta no entra. Sustento, vigencia y segundo dato para coincidir | RF-77, RF-86, RNF-02 |
| **P-02** | Propietario que renta: QR propio a cualquier hora con aviso al arrendatario; no autoriza visitas ni ve la bitácora | RF-84, RF-87, sección 2 |
| **P-03** | Obra y casas deshabitadas: se autoriza desde cualquier lugar; permiso de obra por periodo, resumen diario y aviso inmediato fuera de horario | RF-02, RF-87, RF-88, RF-89 |
| **P-04** | Negocios en viviendas: vivienda con comercio con documento en el panel y aviso agrupado; volumen atípico; renta vacacional como huésped temporal | RF-90, RF-91, RF-92 |
| **R-06** | El guardia indica la apertura manual cuando el operador no da señal | RF-27 |
| **R-07** | Los datos se guardan en la tablet y se envían al recuperar la conexión | RNF-02 |

## 14. Cambios de la versión 0.5

Decisión de dirección: la apertura para residentes es con tag en el parabrisas y con Mi QR; se retira el botón de abrir del teléfono y del reloj.

| Punto | Decisión | Dónde se atiende |
| :--- | :--- | :--- |
| **T-01** | Apertura de residentes con tag en parabrisas: un lector por carril vehicular abre la pluma al detectar un tag vigente; sin botón en la app, en el reloj ni en los widgets. Respaldo: Mi QR y apertura manual del guardia; funciona sin internet con la lista en el controlador | RF-15, RF-16, RF-23, RF-26, RF-31, RF-93 a RF-97, RNF-03, fases y sección 6 |
| **T-02** | Lo que queda fuera de la app: abrir pluma, orden firmada de apertura, geocerca de la entrada y comando de Siri para abrir; la biometría sigue para recurrentes, autorizaciones automáticas, cuenta y tags (RF-67) | RF-23, RF-26, RF-31, RF-52, RF-67, RF-68 |

## 15. Cambios de la versión 0.6

Solicitudes de residentes sobre vigilancia (Luz y Pepe) y decisiones sobre el Centinela.

| Punto | Decisión | Dónde se atiende |
| :--- | :--- | :--- |
| **L-01 a L-03** | Revisión a pedido con fotos/video y dictamen, Modo Vacaciones y revisión inmediata con verificación del Centinela antes de escalar | RF-100, RF-101, RF-102 |
| **P-01 a P-05** | Consulta de rondines, Llegada Segura, contacto con el operador desde la app, aviso de portón abierto o paquete, y vigilancia temporal | RF-103 a RF-107, RF-106, RF-105 |
| **C-04** | El Centinela es de la plataforma (vehículo, software y operación); lo opera el guardia de la caseta y todo va incluido en la cuota mensual | Secciones 2, 4 y 7; RF-108, RF-109, fase 6 |

---
*Versión 0.6 (borrador), basada en las conversaciones del proyecto, la revisión de 36 observaciones, los puntos del panel administrativo, la decisión de usar tags y las solicitudes de residentes sobre vigilancia con Centinelas. Las capacidades de Apple y Google (Siri, Critical Alerts, Assistant/Gemini, APIs de relojes) deben validarse con su documentación vigente. No incluye estimaciones de costo ni tiempo.*
