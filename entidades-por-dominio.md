# Entidades por dominio

Base de datos de la app de acceso y seguridad residencial. Resultado de la Fase 1 (auditoría de contexto), 6 de noviembre de 2026.

Fuentes: requerimientos v0.4, alcance, diccionario de datos (modelo A, 33 tablas), `base-datos/modelo_panel_admin.dbml` (modelo B, 76 tablas) y el API de Laravel. Las claves RF y RNF remiten al documento de requerimientos v0.4.

> Es un mapa conceptual de entidades, no el listado final de tablas. Las tablas, las relaciones y las máquinas de estados se definen en la Fase 2.

## Resumen

| Dominio | Entidades núcleo | RF clave |
|---|---|---|
| 🔐 Identidad | usuario global, credencial, TOTP, sesión, dispositivo | RNF-05, RF-66 |
| 🏘️ Estructura | fraccionamiento, configuración, geocerca, caseta, carril, controlador, tablet, vivienda, comercio | RF-52, RF-60, RF-90 |
| 👥 Personas | residente, solicitud de alta, guardia, turno, administrador, vehículo | RF-61…65, RF-78, RF-84, RF-87 |
| 🎟️ Permisos previos | invitación, acceso recurrente, permiso de servicio, permiso de obra, lista restringida | RF-07, RF-10, RF-55, RF-77, RF-88 |
| 🚗 Accesos (núcleo) | acceso, destino por vivienda, excepción, coincidencia restringida, aviso e intentos, paquete | RF-01…09, RF-70…76, RF-86 |
| 🚧 Aperturas | orden firmada, apertura, solicitud de paso, evento de controlador | RF-20…28, RF-79, RF-80 |
| 🆘 Pánico | alerta, ubicación en vivo, escalamiento | RF-40…46 |
| 📜 Auditoría | acciones de administradores, eventos de seguridad, exportaciones, outbox | RF-51, RF-82, RNF-07 |

## 🔐 Identidad

Es la cuenta global de una persona. No depende del fraccionamiento: su papel en cada uno vive en el dominio Personas.

| Entidad | Qué representa | RF |
|---|---|---|
| **Usuario** | Persona con cuenta: nombre, teléfono o correo verificados. Puede ser residente en un fraccionamiento y administrador en otro. | RF-53, RF-61 |
| **Credencial** | Contraseña del administrador, guardada como hash. | RNF-05 |
| **Factor TOTP** | Segundo factor del administrador y sus códigos de recuperación. | RNF-05 |
| **Sesión** | Sesión de navegador o de app. Se revoca al dar de baja a la persona. | RNF-05, RF-65 |
| **Dispositivo** | Teléfono o reloj de la cuenta, con su llave para firmar acciones sensibles y su token push. Máximo 3 por cuenta. | RF-66, RF-67 |
| **Contacto de emergencia** | A quién se avisa si hay pánico fuera del fraccionamiento. | RF-42, RF-45 |

## 🏘️ Estructura

Es el lugar físico y su equipo. Lo da de alta la administración y casi no cambia.

| Entidad | Qué representa | RF |
|---|---|---|
| **Fraccionamiento** | Raíz del aislamiento: todo lo demás le pertenece. | RF-60, RNF-11 |
| **Configuración** | Tiempos de escalamiento, límites, retención y vigencias del fraccionamiento. | RF-52 |
| **Geocerca** | Perímetro del fraccionamiento y círculo de la entrada. | RF-23, RF-42 |
| **Caseta** | Cada entrada del fraccionamiento. | RF-52 |
| **Carril** | Paso de una caseta: vehicular o peatonal, de entrada o de salida, exclusivo de residentes o compartido. | RF-21, RF-23, RF-52 |
| **Controlador** | Equipo con relevadores que acciona la pluma, con su certificado. | RF-20, RF-54, RNF-03 |
| **Tablet** | Terminal del guardia en la caseta, con su llave para firmar órdenes sin internet. | RF-25 |
| **Vivienda** | Casa o lote al que llegan las visitas. | RF-50, RF-84, RF-88 |
| **Comercio** | Vivienda con negocio, su horario y su documento. | RF-90 |

## 👥 Personas

Es el papel que cada persona tiene dentro de un fraccionamiento.

| Entidad | Qué representa | RF |
|---|---|---|
| **Residente** | Liga entre un usuario y una vivienda, con su rol: titular, integrante, propietario no residente o huésped temporal. | RF-64, RF-87, RF-92 |
| **Solicitud de alta** | Petición de un usuario para unirse a una vivienda, y la decisión de la administración. | RF-61, RF-62, RF-63 |
| **Invitación de residente** | Enlace para incorporarse al hogar. Es distinta de la invitación de visita. | RF-50, RF-64 |
| **Guardia** | Personal de la caseta, con su PIN. | RF-78 |
| **Turno** | Qué guardia estaba en qué tablet y en qué horario. | RF-78, RF-28 |
| **Administrador** | Membresía de un usuario en un fraccionamiento, con su rol y sus permisos. | RF-50, RF-53, RF-77 |
| **Vehículo** | Vehículo de la vivienda, sus conductores y, si aplica, su tag RFID. | RF-23 |

## 🎟️ Permisos previos

Es quién puede entrar por adelantado y quién lo tiene prohibido.

| Entidad | Qué representa | RF |
|---|---|---|
| **Invitación** | Invitación única o de evento creada por un residente, con QR y PIN de respaldo. | RF-07, RF-75 |
| **Invitado de evento** | Cada persona que se registra con el enlace del evento. | RF-75 |
| **Acceso recurrente** | Visita o servicio frecuente, con días, horario y vencimiento. | RF-10, RF-11, RF-12 |
| **Permiso de servicio** | Proveedor de la administración, con vigencia, horario y placas. | RF-55, RF-09 |
| **Permiso de obra** | Obra en una vivienda o lote, con responsable, licencia y horario. | RF-88, RF-89 |
| **Lista restringida** | Persona o placa con acceso restringido, con su sustento y su vigencia. | RF-77, RF-86 |

## 🚗 Accesos (núcleo)

Es quién llegó, a dónde iba, quién lo autorizó y cuándo salió.

| Entidad | Qué representa | RF |
|---|---|---|
| **Acceso** | Cada llegada a la caseta, con su tipo y su estado. | RF-01, RF-70, RF-74, RF-76 |
| **Datos personales del acceso** | Nombre, placa, identificación y foto de la visita, separados para poder purgarlos. | RNF-06, RNF-07 |
| **Destino** | Cada vivienda a la que va un acceso, con su respuesta propia. | RF-06, RF-71 |
| **Excepción** | Caso que decide la administración: apertura manual por excepción o coincidencia con la lista restringida. | RF-22, RF-04, RF-86 |
| **Coincidencia restringida** | Qué dato del acceso coincidió con qué registro de la lista, y si fue exacta o posible. | RF-86 |
| **Aviso** | Mensaje de dominio a un residente: solicitud, llegada, resumen o alerta. | RF-02, RF-04, RF-89, RF-90 |
| **Intento de aviso** | Cada envío por canal: push, WhatsApp, llamada o SMS. | RF-04, RF-83 |
| **Paquete** | Paquetería que se recibe en la caseta y su entrega. | RF-72, RF-73 |
| **Evento de acceso** | Historial inmutable de cada cambio de estado del acceso. | RF-70, RNF-07 |

## 🚧 Aperturas

Es quién levantó la pluma, cuándo y por qué.

| Entidad | Qué representa | RF |
|---|---|---|
| **Orden de apertura** | Orden firmada que expira y no se reutiliza. | RNF-03, RF-80 |
| **Apertura** | Registro de cada levantamiento: origen, motivo, carril y quién lo pidió. | RF-21, RF-22, RF-24, RF-27, RF-28 |
| **Solicitud de paso** | Botón del residente en un carril compartido, que confirma el guardia. | RF-23 |
| **Consumo de PIN** | Cada uso del PIN de una invitación o de un acceso recurrente, para respetar el límite diario. | RF-07, RF-11 |
| **Evento de controlador** | Señales de la pluma: vehículo detectado, vehículo que pasó, apertura física y posición. | RF-27, RF-79 |

## 🆘 Pánico

Es quién pidió ayuda y dónde estaba.

| Entidad | Qué representa | RF |
|---|---|---|
| **Alerta** | Pánico o caída, y quién la atendió. | RF-40, RF-44, RF-46 |
| **Ubicación de alerta** | Ubicación en vivo mientras la alerta está activa. | RF-42 |
| **Escalamiento** | A quién se escaló la alerta si nadie la atendió. | RF-44, RF-45 |

## 📜 Auditoría

Es qué cambió, quién lo cambió y qué salió del sistema.

| Entidad | Qué representa | RF |
|---|---|---|
| **Acción de administrador** | Bitácora inmutable de altas, bajas, aprobaciones, permisos y configuración. | RF-82 |
| **Evento de seguridad** | Inicios de sesión, cambios de contraseña, 2FA y concesiones de plataforma. | RNF-05 |
| **Exportación** | Cada exportación de la bitácora: quién la pidió y qué incluía. | RF-51 |
| **Evento de salida (outbox)** | Cambio de dominio pendiente de publicar a notificaciones, tiempo real o controlador. | RNF-01 |
