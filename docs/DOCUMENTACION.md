# DX Agent — Guía de uso

*Creado por Diego – LW5HR, con Franco – LU2PWY, Juan Manuel – LU9HVR, Gustavo – LU7HW, Daniel – LU1HLH. Licencia MIT. Apoyá DX Agent: [Ko-fi](https://ko-fi.com/lw5hr) · [PayPal](https://paypal.me/diegodimunzio1).
English version: [DOCUMENTATION.md](DOCUMENTATION.md).*

DX Agent vigila el DX cluster y la Reverse Beacon Network (RBN), compara cada
estación reportada con tu log ADIF y avisa cuando sale al aire una que todavía
necesitás. Funciona en tu PC, con un panel en el navegador, y solo requiere Python
3.11 o posterior: no hay nada más que instalar.

Responde tres preguntas en tiempo real:

- **¿Me falta esta estación en el log?** Un país nuevo (ATNO), una banda nueva, un
  modo nuevo o un indicativo de tu lista de seguimiento.
- **¿Es una DXpedición activa y realmente la necesito?** Sigue las listas de
  DXpediciones de NG3K, 425 DX News y Club Log e indica si trabajarla suma algo nuevo, te da una
  confirmación que falta o no suma nada.
- **¿Voy a obtener la confirmación?** Muestra si la estación sube sus QSOs a LoTW y
  puede descargar tus propias confirmaciones de LoTW.

Contenido:

1. [Inicio rápido](#1-inicio-rápido)
2. [Panel web](#2-panel-web)
3. [Página de configuración](#3-página-de-configuración)
4. [Comandos](#4-comandos)
5. [Cómo leer las alertas](#5-cómo-leer-las-alertas)
6. [El archivo de configuración](#6-el-archivo-de-configuración)
7. [Integración con LoTW](#7-integración-con-lotw)
8. [Cómo funciona](#8-cómo-funciona)
9. [Archivos del proyecto y datos descargados](#9-archivos-del-proyecto-y-datos-descargados)
10. [Limitaciones conocidas y solución de problemas](#10-limitaciones-conocidas-y-solución-de-problemas)

---

## 1. Inicio rápido

**Lo más fácil:** descomprimir `dx-agent-<versión>-windows.zip` en cualquier carpeta y
hacer doble clic en **start-dx-agent.bat**. Ese zip ya trae Python, así que no hay que
instalar nada. DX Agent corre sin ventana: el panel se abre en el navegador,
**Detener** (arriba a la derecha del panel) lo cierra, y si se hace doble clic de nuevo
mientras corre, solo abre el panel. Si no puede arrancar, un cartel de Windows dice por
qué; lo que va mostrando queda en `data/dx-agent.log`, y **start-dx-agent-console.bat**
lo corre en una ventana para verlo. (El zip con el código, `dx-agent-<versión>-source.zip`,
requiere
[Python 3.11+](https://www.python.org/downloads/) instalado con "Add python.exe to
PATH"; si no está, start-dx-agent.bat lo avisa y ofrece instalarlo con winget.) El primer inicio crea `config.toml` a partir
de `config.example.toml`; después configurar indicativo y log en **Configuración**.
Desde una terminal:

1. Abrir una terminal. En VS Code: **Terminal → New Terminal**.
2. Ir a la carpeta del proyecto:

   ```powershell
   cd C:\path\to\dx-agent
   ```

3. Iniciarlo:

   ```powershell
   python -m dxagent run
   ```

   El navegador abre el panel en **http://localhost:8073**.
4. La primera vez, hacer clic en **Configuración** (arriba a la derecha) y revisar el
   indicativo, la ruta del log ADIF y, si se quiere, la contraseña de LoTW. Luego
   **Guardar y aplicar**. (Si el panel aparece en inglés, el botón **EN** del
   encabezado lo pasa a español.)
5. Para detenerlo, presionar **Ctrl+C** en la terminal (muestra `73!`).

Hay que ejecutarlo desde la carpeta `dx-agent`, porque ahí busca `config.toml`. Desde
cualquier otra carpeta, indicar la ruta de la configuración:

```powershell
python -m dxagent -c C:\path\to\dx-agent\config.toml run
```

Un arranque normal se ve así en la terminal, seguido de una línea por cada estación
necesitada:

```text
[log] 23817 QSOs, 271 DXCC entities (248 confirmed) from N0CALL_20261013_175723_backup.adi (1.6s)
[lotw] no LoTW password set - using confirmations from the ADIF log only
[dxped] NG3K 69, 425 DX News 71, Club Log 3 -> 106 DXpeditions
[dxped] 36 active now
[agent] alerting on: atno, dxpedition, new_band, new_mode, watchlist | bands: 160m ... 6m | modes: CW PHONE DIGI
[ui] dashboard at http://localhost:8073
[rbn-cw] connected to telnet.reversebeacon.net:7000
[rbn-ft8] connected to telnet.reversebeacon.net:7001
[cluster] connected to w3lpl.net:7373
[cluster] logged in as N0CALL
```

Cada 10 minutos imprime una línea de estado con la cantidad de spots procesados y de
alertas generadas.

**Notificaciones:** las alertas también aparecen como notificaciones de Windows
(Windows 10/11), sin instalar nada adicional.

---

## 2. Panel web

`run` inicia el panel y lo abre en el navegador en **http://localhost:8073**. La
terminal sigue funcionando igual que antes; el panel es otra vista del mismo agente y
se cierra al detener `run`.

```powershell
python -m dxagent run               # terminal + panel, abre el navegador
python -m dxagent run --no-browser  # panel activo, sin abrir una pestaña
python -m dxagent run --no-ui       # solo terminal
```

### Qué hay en la pantalla

| Zona | Qué muestra |
|------|-------------|
| **Encabezado** | Tu indicativo; una pastilla por fuente de spots (verde = conectada, ámbar = conectando, rojo = caída) con la cantidad de spots (al pasar el mouse muestra el servidor y la hora del último spot); los totales de tu log; DXpediciones activas; botones **Configuración**, **Acerca de** (qué es DX Agent, quién lo hizo, los colaboradores, cómo apoyarlo, ayuda), **ES/EN** (idioma), **Tema** y **Sonido**; reloj UTC. `#about` abre la ventana Acerca de |
| **Filtros** | Botones por tipo de alerta, banda y modo, y un cuadro de búsqueda por indicativo o país. Sin ningún botón seleccionado se muestra todo. Una alerta cuenta para cada uno de sus motivos, así que una DXpedición en tu lista de seguimiento aparece tanto en **DXPED FALTA** como en **SEGUIMIENTO**; **SEGUIMIENTO** además muestra los spots de tus indicativos en seguimiento que no dieron alerta (filas grises **SPOT**), para que veas cuándo están en el aire. **Ocultar confirmados** saca las estaciones cuya entidad ya tenés confirmada en esa banda y modo (una alerta de indicativo nuevo, una fila SPOT): quedan solo las que suman a tu log. El navegador recuerda los filtros |
| **Tabla de alertas** | Una fila por alerta, la más nueva arriba, coloreada según el tipo: UTC, alerta, indicativo, kHz, banda, modo, país, qué te falta, LoTW, **cerca tuyo** (ver más abajo), quién la escuchó. Las filas nuevas destellan; se atenúan a los 10 minutos y más a los 30 (al pasar el mouse se ve la antigüedad) |
| **Pestaña Estación** | Al hacer clic en una alerta, o escribir un indicativo en el cuadro, muestra la grilla banda × modo de ese país (C verde = confirmado, W ámbar = trabajado, gris = no trabajado), si trabajaste ese indicativo exacto, el estado de LoTW, los datos de la DXpedición, quién lo escucha más cerca tuyo, un botón **Consultar PSKReporter**, los botones **Ver solo** este indicativo y **☆ seguimiento**, sus últimos spots y enlaces a QRZ.com, HamQTH y DXWatch |
| **Pestaña DXpediciones** | DXpediciones activas y las que empiezan en los próximos 30 días. Etiqueta: **ATNO**, **No confirmado** o cuántas bandas tenés (por ejemplo "1 de 10 bandas", resaltada cuando son menos de la mitad). Marcar **solo las que faltan** oculta las que ya tenés en la mayoría de las bandas. Ordenadas por las que están **en el aire** (spoteadas en los últimos 30 minutos, la más reciente arriba), por el puesto en el **Most Wanted** de Club Log (el país más buscado primero) o por fecha de inicio: **Orden**, arriba. Cada tarjeta muestra sus fuentes y el puesto en el **Most Wanted** de Club Log; las spoteadas en los últimos 30 minutos se marcan en verde como **EN EL AIRE**. Al hacer clic se abre su grilla; la **☆** la agrega a tu lista de seguimiento |

**Colores de banda.** Cada banda de HF y 6 m tiene su color, que se usa en la etiqueta
de banda de la tabla, en los filtros de banda, en la grilla de la estación y en la
configuración: 160 ámbar, 80 violeta, 60 rojo, 40 azul, 30 oliva, 20 rosa, 17 verde,
15 turquesa, 12 naranja, 10 azul marino, 6 marrón (4 m, 2 m y 70 cm comparten el gris).
El orden se verificó con un simulador de daltonismo para que las bandas vecinas se
distingan bien, y la etiqueta siempre muestra el nombre de la banda, así que el color
nunca es la única pista.

**ES/EN** cambia el idioma del panel entre español e inglés. La primera vez usa el
idioma del navegador; después recuerda la elección. Un enlace también puede
forzarlo: `http://localhost:8073/?lang=es`. Los nombres de países y las notas de las
DXpediciones quedan en inglés, porque vienen así de sus fuentes.

**Tema** alterna entre **Auto** (sigue el modo claro/oscuro de Windows), **Claro** y
**Oscuro**. El navegador recuerda la elección. Un enlace también puede forzar uno:
`http://localhost:8073/?theme=light`.

**Sonido** emite un pitido en la pestaña del navegador para las alertas ATNO
y de prioridad 2. Está apagado por defecto; el agente en sí no hace ningún sonido.

**Enlaces para guardar como favoritos:** `http://localhost:8073/#settings` abre la
configuración, `#call=VP8PJ` abre esa estación y `#dxped` abre la pestaña de
DXpediciones.

**Cuando algo falla** (por ejemplo, no encuentra el archivo del log), el agente
mantiene el panel funcionando y muestra una franja amarilla con el motivo y un enlace
a la configuración, para corregirlo sin usar la terminal.

### Sintonizar la radio

Con una radio configurada (Configuración → **Radio (botón Sintonizar)**), al hacer
clic en la frecuencia de una alerta se le pide a tu programa de log o de control de
radio que mueva la radio a esa frecuencia, y la estación se abre a la derecha, igual que al
hacer clic en la fila. La pestaña Estación también tiene un botón
**Sintonizar**: para la fila en la que hiciste clic en la lista (resaltada más fuerte que
las otras filas del mismo indicativo) o, si no, para el último spot de esa estación. Un mensaje arriba a la derecha
indica si funcionó.

| Destino | Cómo se comunica DX Agent | Puerto por defecto | ¿También el modo? |
|---------|---------------------------|--------------------|-------------------|
| Log4OM 2 | Control remoto por UDP (`SetTxFrequency`, `SetCallsign`); el modo, por OmniRig | 2241 | sí, por OmniRig (el `SetMode` de Log4OM solo cambia su pantalla) |
| N1MM Logger+ | UDP `radio_setfrequency` | 13064 | no (N1MM no tiene campo de modo) |
| OmniRig | COM de Windows, mediante PowerShell; radio 1 o 2 | — | sí |
| Hamlib rigctld | TCP `F <Hz>` / `M <modo> -1` | 4532 | sí |
| flrig | XML-RPC `rig.set_frequency` / `rig.set_mode` | 12345 | sí |
| DXLab Commander | TCP `CmdSetFreqMode` | 52002 | sí |

- **Modo elegido:** CW para spots de CW; USB o LSB para fonía (LSB por debajo de
  10 MHz, salvo 60 m); RTTY para RTTY; para FT8/FT4, USB con OmniRig (como WSJT-X usa
  la radio) o el modo de datos con Hamlib, flrig y DXLab. Desmarcar **También cambiar
  el modo** para cambiar solo la frecuencia.
- **Si la radio ya está en un modo que sirve, se mantiene (OmniRig, y Log4OM a través de
  OmniRig):** para un spot de FT8/FT4, una radio en USB o en su modo de datos solo cambia de
  frecuencia; para CW, una en CW o CW-R. Así una radio que necesita su modo de datos para FT8
  (un IC-9100 en USB-D) lo conserva de un spot de FT8 al siguiente: se pone en USB-D una vez.
  Los spots de fonía siempre ponen USB o LSB, para que nunca quede un modo de datos con el
  micrófono silenciado.
- **Log4OM y el modo:** el control remoto de Log4OM acepta `SetMode`, pero solo cambia el modo
  en la pantalla de Log4OM, no el de la radio (probado: Log4OM 2.42 con un IC-9100). Log4OM
  maneja la radio a través de OmniRig, así que con **También cambiar el modo** marcado DX Agent
  pone frecuencia y modo por OmniRig, en la radio de OmniRig que está en línea (primero prueba
  la de **Radio de OmniRig**), y Log4OM sigue a la radio como siempre; el indicativo se le sigue
  mandando a Log4OM. Sin OmniRig, o sin ninguna radio en línea, Sintonizar mueve la frecuencia
  por Log4OM y el mensaje avisa que el modo no cambió.
- **Log4OM:** activar Settings → Software integration → Connections → **Remote
  control** (puerto 2241). El documento de Log4OM escribe
  el comando `SetTxFrequeny`; DX Agent usa la forma que se informó que funciona,
  `SetTxFrequency`. También manda el indicativo del spot (`SetCallsign`), así la ventana de
  QSO de Log4OM queda lista para loguearlo; si escribiste un indicativo ahí a mano, Log4OM no
  lo pisa (es su regla).
- **Solo desde el panel:** igual que guardar la configuración, Sintonizar solo
  funciona desde la propia página del panel, así que otros sitios web no pueden mover
  tu radio.
- **Probado:** todos los destinos con programas de prueba; Log4OM 2.42 con una radio
  real (la frecuencia funciona, el modo no).

**WSJT-X.** Configuración → Radio → marcar **Mandar también el indicativo a WSJT-X**, y en
WSJT-X marcar Settings → Reporting → **Accept UDP requests**. Después, Sintonizar en un spot
de FT8/FT4 pone la radio en la frecuencia de dial de FT8/FT4 (14074, no la 14075.6 del
spot) y completa en WSJT-X el **DX Call** y el **DX Grid** (si la RBN lo informó), elige FT8
o FT4 y mueve la frecuencia de Rx al audio de la estación (1600 Hz para 14075.6), y genera
los mensajes estándar: vos hacés clic en **Enable Tx**. Funciona con cualquiera de los
controles de radio de arriba o solo (control de radio: ninguno). DX Agent encuentra
WSJT-X solo (no hay que configurar puerto) y no ocupa el puerto UDP 2237 de WSJT-X, así
que Log4OM, JTAlert o GridTracker siguen funcionando.

### Seguir una estación

En la pestaña Estación (clic en una alerta, o en una DXpedición de la pestaña DXpediciones):

- **Ver solo VP8PJ** deja en la lista y en el mapa solo ese indicativo. La lista muestra
  también sus spots que no generaron alerta (filas grises **SPOT**, por ejemplo en una
  banda que ya tenés), así una DXpedición en el aire nunca aparece con la lista vacía. Arriba de la
  lista aparece el chip **Solo VP8PJ ✕**; hacé clic en él (o en **Ver todos**, o en
  **Limpiar filtros**) para volver a ver todos. Este filtro no se guarda: al recargar
  la página se ve todo.
- **☆ Agregar a seguimiento / ★ Quitar de seguimiento** suma el indicativo a tu lista de
  seguimiento o lo saca. Se guarda en `config.toml` y vale al instante, sin reiniciar.
  Si el indicativo entra por un comodín como `VP8*`, se ve **★ En seguimiento (VP8*)**;
  los comodines se editan en Configuración. La pestaña DXpediciones tiene la misma
  estrella en cada tarjeta.
- **Últimos spots** lista los spots de ese indicativo de los últimos 60 minutos, del
  cluster y de la RBN, incluso los que no generaron alerta (por ejemplo una banda que
  ya tenés). Se guardan para DXpediciones, indicativos en seguimiento e indicativos que
  dieron alerta, desde que arrancó el agente. Con el control de radio activo, hacé clic
  en una frecuencia para sintonizarla.

### Activaciones: POTA, SOTA, IOTA...

El menú **Actividad** (después de los botones de modo) muestra solo activaciones: **POTA,
SOTA, IOTA, WWFF, BOTA, COTA (castillos WCA), LOTA (faros), TOTA, ROTA, JOTA, YOTA** y
**EME**, o *cualquier activación*. Al lado de cada una, cuántas estaciones hay en la última hora.

- La lista muestra sus alertas y también sus spots que no generaron alerta (filas grises
  **SPOT**: un parque en un país que ya tenés también aparece), de la última hora.
- Cada fila dice la activación debajo de *Qué necesitás*, con su referencia: `POTA US-5584`,
  `IOTA EU-038`, `WWFF KFF-6494`...
- **POTA y WWFF:** DX Agent lee además sus propias listas de quién está activando ahora
  (pota.app y wwff.co, cada 2 minutos). Cualquier spot de esas estaciones lleva su parque,
  incluso un spot de RBN de FT8 (RBN descarta el "POTA" de "CQ POTA"), y los activadores
  spoteados solo en esos sitios también aparecen ("(POTA)" en *Escuchado por*). La lista de
  SOTA de quién está en el aire pide registrarse en SOTA y una aprobación, así que SOTA se
  reconoce solo por el comentario.
- **Cumbres SOTA:** DX Agent guarda la lista de cumbres de SOTA (storage.sota.org.uk, se baja
  de nuevo cuando tiene un mes). Una referencia de cumbre cuenta solo si es una cumbre válida,
  salvo que quien la spoteó haya escrito "SOTA" antes (una cumbre más nueva que nuestra copia).
  Pasando el mouse sobre la activación se ve el nombre de la cumbre, su altura y sus puntos; los
  parques de POTA y WWFF muestran ahí su nombre también.
- **En el mapa:** las activaciones van en su cumbre (un triángulo) o su parque (un cuadrado),
  según la lista de SOTA y las de POTA y WWFF; con el color de su alerta, grises si no generaron
  ninguna.
- **Cumbres y parques nuevos:** con una Actividad elegida, una cumbre, parque, isla (IOTA) o
  castillo que no trabajaste **en esa banda** se marca *nueva*, aunque tengas el país confirmado:
  un spot que no generó alerta aparece como **REF NUEVA** ("referencia no trabajada en 20m"), la
  referencia dice *nueva*, el mapa la pinta como BANDA NUEVA y *Ocultar confirmados* no la
  esconde. Sin Actividad elegida no cambia nada. Lo que trabajaste sale de:
  - tu log, cuando el QSO dice la referencia: `SOTA_REF`, `POTA_REF` (un two-fer cuenta para
    cada parque), `WWFF_REF`, `IOTA`, `SIG` + `SIG_INFO`, y las referencias de premios de Log4OM;
  - lo que DX Agent vio en el aire: recuerda dónde estuvo cada activador y cuándo (los últimos
    32 días, en `data/dxagent.db`). Un QSO de tu log con esa estación a menos de 45 minutos
    de uno de sus spots cuenta para la cumbre o el parque en el que estaba, aunque tu programa de
    log no la haya anotado (con `/P` o sin). Esos QSOs se guardan para siempre en
    el mismo archivo. Solo conoce las activaciones que DX Agent vio: se completa
    de ahora en adelante.
- Los clusters no tienen un campo para esto: DX Agent lee el comentario del spot. Reconoce
  el nombre del programa (`POTA`, `WWFF`, `WCA`, `ILLW`, `JOTA`, `EME`...) y las referencias por
  su formato (`US-5584`, `KFF-6494`, `EU-038`, `LA/VF-010`, `DL-01121`). Un spot en el que quien
  lo publicó no escribió ninguna de las dos cosas no se reconoce como activación. Las
  estaciones con YOTA en el indicativo (GB23YOTA) cuentan como YOTA.
- Como los demás filtros, se guarda; **Limpiar filtros** lo apaga.

### Mapa mundial

Hacé clic en **Mapa** (al lado de **Lista**, a la izquierda de los filtros) para ver las
mismas alertas en un mapa del mundo. Los filtros valen para las dos vistas.

- **Los puntos** son alertas, con el color de su tipo. La posición de la estación sale del
  grid locator que informa la RBN en los spots de FT8/FT4; si no, es el centro del país
  (marcado "aprox." en el detalle). Pasá el mouse para ver los datos y hacé clic para
  seleccionarla.
- **Los rombos** son DXpediciones activas: rojo = ATNO, ámbar = sin confirmar, turquesa = tenés
  el país pero en menos de la mitad de tus bandas, gris = ya la tenés. Si tienen una alerta de
  los últimos 30 minutos, el rombo toma el color de la alerta. Un **borde verde** quiere decir
  que la spotearon en los últimos 30 minutos (está en el aire). La leyenda abajo a la izquierda
  lo resume.
- **Punto azul:** tu QTH, según tu grid locator.
- **Zona sombreada:** donde es de noche ahora (su borde es la línea gris); se actualiza cada minuto.
- **Estación seleccionada:** la ruta de círculo máximo punteada desde tu QTH, el **rumbo
  de antena** (y el paso largo) y la distancia, y puntitos para cada receptor que la
  escuchó en los últimos 15 minutos (verde = cerca tuyo).
- **Zoom** con la rueda del mouse o los botones **+ / −**, **arrastrar** para moverse y
  **⟲** para ver el mundo entero. Al hacer zoom aparecen los indicativos de todas las estaciones;
  donde se pisarían, se muestran los más importantes (pasá el mouse por un punto para ver su indicativo).
- La pestaña Estación también muestra el **rumbo**, grande, con una brújula que apunta hacia
  ahí, y debajo el paso largo y la distancia, en cualquiera de las dos vistas.
- Las fronteras son las **DXCC**, no solo las de los países: también están separadas la Rusia
  europea y la asiática, Inglaterra, Escocia y Gales o el Monte Athos. Pasá el mouse por el
  mapa para ver qué entidad es. Cada entidad muestra su prefijo cuando entra en pantalla
  (hacé zoom para ver los de las chicas).
- Las costas y las fronteras de los países salen de Natural Earth 1:50m (dominio público); las
  entidades DXCC y las fronteras que los países no tienen, del mapa DXCC de GridTracker
  (licencia BSD). Se descargan la primera vez que abrís el mapa y se guardan en
  `data/world-countries-50m.json` y `data/dxcc-shapes.json`.
- Un link directo al mapa: `http://localhost:8073/?view=map`.

#### Tu DXCC en el mapa

El menú **DXCC**, arriba a la izquierda del mapa, pinta cada entidad según cómo vas en un
diploma: **verde** = confirmada, **ámbar** = trabajada pero sin confirmar, **rojo** = no
trabajada. Al lado dice cuántas tenés confirmadas y trabajadas, sobre las entidades actuales.
Pasá el mouse por una entidad para ver su estado. Elegí el diploma en el primer menú y, para
Mixto, Fonía, CW o Digital, una banda en el segundo:

- **Mixto**, **Fonía**, **CW**, **Digital** con *todas las bandas*: esos diplomas;
- **Mixto** y una banda: el diploma de esa banda (de 160 a 6 m siempre; 2 m, 70 cm y más
  arriba cuando las trabajaste);
- **Fonía**, **CW** o **Digital** y una banda, por ejemplo **CW** y **40 m**: tus slots en esa
  banda y modo. No es un diploma de la ARRL, pero así se persiguen los slots;
- **Satélite**;
- **Challenge (160–6 m)**: cuantas más bandas confirmadas, más verde (de 0 a 10); el total
  cuenta banda-entidad, como el DXCC Challenge.

Sigue las reglas del DXCC de la ARRL (arrl.org/dxcc-rules):

- Confirmada quiere decir **LoTW o QSL en papel** recibida (QSL_RCVD); eQSL no cuenta. Una
  tarjeta recibida no está acreditada hasta que la revisa la ARRL o un card checker, así que
  tus totales en la ARRL pueden ser menores.
- Los QSOs por satélite (PROP_MODE SAT) cuentan solo para Satélite; los de repetidoras e
  internet (RPT, INTERNET, ECH, IRL), los de 60 m y las estaciones en barcos o aviones (/MM,
  /AM) no cuentan; los QSOs anteriores al 15/11/1945 no cuentan, y el CW anterior a 1975
  cuenta solo para Mixto.
- Solo cuenta la lista DXCC actual: las entidades eliminadas quedan afuera.
- No revisa desde dónde operaste ni con qué indicativo: cuenta todo QSO de tu log.

Las entidades chicas (islas, microestados, enclaves como el Vaticano o Guantánamo) llevan además
un halo de su color, para verlas sin hacer zoom.

#### Tu WAZ en el mapa

Elegí **WAZ** en el primer menú (en lugar de DXCC) para pintar las 40 **zonas CQ** según
cómo vas en el Worked All Zones de CQ: el mismo verde / ámbar / rojo, cada zona con su número
y sus bordes. Pasá el mouse por una zona para ver su nombre y su estado. Elegí el diploma:

- **Mixto, SSB, CW, RTTY, AM**; **SSTV** (QSOs desde 1973); **Digital** (desde 2000, sin
  RTTY, que tiene su propio diploma);
- un modo y una banda, por ejemplo **CW** y **40 m** (80-10 m desde 1973; 30, 17 y 12 m desde
  1991), o **Mixto** y una banda (160 m desde 1975, 30 zonas para pedirlo; 6 m, 25 zonas);
  de 2 m para arriba cuentan solo los QSOs por EME;
- **Satélite** (desde 1989, 25 zonas) y **EME** (desde 1973, 25 zonas);
- **5BWAZ**: las 40 zonas en cada una de 80, 40, 20, 15 y 10 m, cualquier modo, QSOs desde
  1979: cuantas más bandas confirmadas, más verde; el total cuenta banda-zona (200; el
  diploma básico es con 150).

Sigue las reglas de CQ (n4baa.com/9_Rules.pdf, agosto 2026):

- Confirmada quiere decir **LoTW, QSL en papel o eQSL** (WAZ acepta eQSL; DXCC no).
- La zona es la que tu programa de log guardó con el QSO (CQZ en el ADIF) o, si falta, la de
  la entidad según cty.csv. En países con varias zonas (EE. UU., Canadá, Rusia, Brasil, China,
  Australia...) revisá que tu programa la complete.
- Barcos y aviones (/MM, /AM), repetidoras e internet nunca cuentan; los QSOs anteriores al
  14/11/1945 tampoco.
- No se controla que hayas operado desde una sola entidad DXCC, ni las categorías
  Tradicional / Remota.

**Zonas CQ** (la casilla del menú) muestra los bordes y números de las zonas en cualquier
vista, con DXCC o sin nada.

### Seguridad

- El panel solo escucha en `127.0.0.1`, así que otras computadoras no pueden
  acceder.
- Solo responde a pedidos dirigidos a `localhost` (protección contra trucos de
  *DNS rebinding* desde sitios web).
- La configuración solo se puede guardar desde la propia página del panel. Otros
  sitios web que visites no pueden enviar un pedido que la modifique ni mostrar el
  panel dentro de su propia página.
- El texto de los spots (los comentarios del cluster los escribe cualquiera en
  internet) siempre se muestra como texto plano, nunca se ejecuta como código.

---

## 3. Página de configuración

Hacer clic en **Configuración** en el encabezado (o abrir
`http://localhost:8073/#settings`). Cambiar lo necesario y hacer clic en **Guardar y
aplicar**: el agente reescribe `config.toml`, vuelve a cargar el log y reconecta las
fuentes en pocos segundos. No hace falta reiniciarlo.

| Sección | Qué se puede configurar |
|---------|-------------------------|
| **Estación** | Tu indicativo; **Tu log**: un archivo ADIF de tu logger, o **Solo LoTW** (ver LoTW como log); la ruta del log ADIF (una por línea; `*` elige el archivo más nuevo, por ejemplo `…/CALL_*_backup.adi` para los backups de Log4OM). **Buscar…** abre la ventana de archivos de Windows para elegirlo (si elegís un backup de Log4OM con fecha, se convierte en `…/CALL_*_backup.adi` para usar siempre el más nuevo). **Probar** muestra qué archivo encontró, su tamaño, fecha y cantidad de QSOs; tu grid locator (vacío = se toma de tu log) |
| **LoTW** | Activar o desactivar la descarga de confirmaciones; usuario de LoTW (vacío = tu indicativo); contraseña de LoTW; cada cuánto descargar. **Probar ingreso** prueba el usuario y la contraseña con LoTW |
| **Alertas** | Qué tipos de alerta querés; bandas; modos; cuándo una DXpedición o un indicativo de la lista de seguimiento cuenta como trabajado; **Lista de seguimiento**; **Ignorar**; cuánto esperar antes de repetir la misma alerta; el radio de "cerca tuyo" y **Avisar solo si se escucha cerca tuyo**; notificaciones de Windows y para qué alertas |
| **Fuentes de spots** | DX cluster activado o no, sus servidores (`servidor:puerto`, en orden) y comandos al conectar; **Spots recientes al conectar** (minutos, 0 = no); de qué continentes tomar los spotters del cluster; RBN activada o no, fuentes CW/RTTY y FT8/FT4; de qué continentes cuentan los skimmers |
| **Radio (botón Sintonizar)** | Qué programa sintoniza la radio (Desactivado, Log4OM 2, N1MM Logger+, OmniRig, Hamlib rigctld, flrig, DXLab Commander), su dirección y puerto (0 = por defecto), si también cambia el modo, y qué radio de OmniRig |
| **DXpediciones, log y panel** | Mostrar DXpediciones que ya tenés; fuentes de DXpediciones y cada cuánto releerlas; cada cuánto revisar el archivo del log; puerto del panel; abrir el navegador al iniciar |

Cómo se comporta:

- **Revisión antes de guardar.** Un indicativo que no parece válido, una ruta de log
  vacía, ninguna banda elegida, un servidor de cluster sin `:puerto` y errores
  similares se listan en rojo, y no se guarda nada hasta corregirlos.
- **Copia de respaldo.** El archivo anterior se conserva como `config.toml.bak`.
- **La contraseña de LoTW** se guarda cifrada para tu usuario de Windows en
  `data/lotw.secret`, nunca en `config.toml`, y no se vuelve a mostrar. El cuadro
  indica si hay una contraseña guardada. Si se deja vacío, se conserva la guardada;
  marcando **Olvidar la contraseña guardada** se elimina.
- **El puerto del panel** cambia recién la próxima vez que se inicie DX Agent.
- **Comentarios.** Al guardar, `config.toml` se reescribe con sus propios
  comentarios breves; los comentarios agregados a mano no se conservan.

---

## 4. Comandos

Todos los comandos se ejecutan desde la carpeta del proyecto como
`python -m dxagent <comando>`. `run` es el comando por defecto.

| Comando                 | Qué hace                                                                          |
|-------------------------|-----------------------------------------------------------------------------------|
| `run`                   | Vigila el cluster y la RBN, sirve el panel y avisa de las estaciones necesitadas hasta Ctrl+C |
| `dxpeditions --days 30` | Lista las DXpediciones activas y las que empiezan en los próximos N días, con tu estado y LoTW para cada una |
| `check INDICATIVO`      | Muestra la grilla trabajado/confirmado del país de ese indicativo, por banda y modo |
| `stats`                 | Cuenta los países trabajados y confirmados por banda y por modo                   |

Para ejecutar las pruebas:

```powershell
python -m unittest discover -s tests -t .
```

### Ejemplo de `check`

`C` = confirmado (LoTW o QSL en papel), `W` = trabajado pero no confirmado,
`.` = no trabajado.

```text
VP2MAW: Montserrat (DXCC 96, NA, CQ 8) - no LoTW
          160m   80m   40m   30m   20m   17m   15m   12m   10m    6m
      CW     .     .     .     .     C     C     W     .     C     .
   PHONE     .     .     .     .     C     .     .     .     .     .
    DIGI     .     .     .     .     .     .     C     .     .     .
  C = confirmed (LoTW/QSL)   W = worked, not confirmed   . = not worked
  this call: not in log
```

### Columna de estado de `dxpeditions`

- `NEED: ATNO`: país nunca trabajado.
- `NEED: not confirmed`: trabajado, pero sin confirmación en ninguna banda.
- `have: 4 bnd, 2 cfm`: trabajado en 4 bandas, confirmado en 2. `(wkd 15,20)` indica
  que ya trabajaste ese indicativo de la DXpedición en esas bandas.

La columna LOTW muestra la fecha de la última subida de la estación, `LoTW announced`
(el listado de la DXpedición dice QSL vía LoTW pero todavía no subió nada) o `no LoTW`.

### Ejemplo de `stats`

```text
        worked   cfm
  160m      58    41
   80m     121    97
   40m     203   181
   ...
    CW     247   226
 PHONE     219   190
  DIGI     232   208
 total     271   248
```

---

## 5. Cómo leer las alertas

Cada alerta es una fila en el panel y una línea en la terminal: hora (UTC), título,
indicativo, frecuencia (kHz), banda, modo, país y luego detalles separados por `|`.
Con el panel en español los títulos y el detalle aparecen traducidos (por ejemplo
**BANDA NUEVA**, "banda nueva"); la terminal y las alertas guardadas en `data/dxagent.db`
siempre usan los nombres en inglés.

```text
14:56Z DXPED NEED  VP2MAW   18095.0   17m FT8   Montserrat   NEED: 17m DIGI never worked | until 10-20 | no LoTW | rbn NG7M -14dB
14:55Z DXPED HAVE  VP2MAW   14030.0   20m CW    Montserrat   have it: 20m CW confirmed | until 10-20 | no LoTW | rbn WE9V 7dB 27wpm
14:55Z NEW BAND    SV2GNJ   18100.0   17m FT8   Greece       LoTW 2022-05-10 | rbn WZ7I -8dB
```

- **El modo** viene de la RBN o del comentario del cluster. Cuando hay que deducirlo
  del plan de bandas se muestra con `?`, por ejemplo `CW?`.
- **El último detalle:** `rbn SPOTTER SNR WPM` para los spots de skimmers, o
  `de SPOTTER: comentario` para los del cluster.
- **Los corchetes** al final listan todos los motivos cuando hay más de uno, por
  ejemplo `[new_band, new_mode]`.

### Tipos de alerta

| En el panel (ES)  | En la terminal / EN | Significado                                                        | Por defecto |
|-------------------|---------------------|--------------------------------------------------------------------|-------------|
| ATNO              | ATNO                | País nunca trabajado                                               | activada    |
| DXPED FALTA       | DXPED NEED          | DXpedición activa que te suma un país, banda, modo o combinación banda+modo nuevos | activada |
| DXPED CONFIRMAR   | DXPED CFM           | DXpedición activa de un país que trabajaste pero no confirmaste en esta banda o modo | activada |
| DXPED YA TENGO    | DXPED HAVE          | DXpedición activa que ya tenés confirmada en esta banda y modo (gris, sin notificación) | activada |
| BANDA NUEVA       | NEW BAND            | País trabajado, pero no en esta banda                              | activada    |
| MODO NUEVO        | NEW MODE            | País trabajado, pero no en este grupo de modos (CW, FONÍA, DIGI)   | activada    |
| SEGUIMIENTO       | WATCHLIST           | Un indicativo de tu lista de seguimiento, todavía no trabajado     | activada    |
| BANDA+MODO        | NEW SLOT            | País no trabajado en esta combinación de banda y modo              | desactivada |
| INDICATIVO NUEVO  | NEW CALL            | Este indicativo exacto no está en tu log (genera muchísimas alertas) | desactivada |

"Por defecto" se refiere a **Avisarme de** en la página de configuración
(`alert_on` en el archivo).

### De dónde salen las DXpediciones

Se unen tres fuentes públicas por indicativo (se eligen en Configuración →
**Fuentes de DXpediciones**; las tres vienen activadas):

| Fuente | Qué aporta |
|--------|------------|
| **NG3K ADXO** | El calendario más conocido de operaciones anunciadas, con fechas, ruta de QSL y notas |
| **425 DX News** | El calendario del boletín semanal italiano: muchas operaciones que NG3K todavía no lista. Quedan afuera los indicativos especiales y de aniversario |
| **Club Log** | Las expediciones que están subiendo su log a Club Log ahora (un log actualizado en los últimos 3 días cuenta como en el aire), el link para buscar tus QSOs en el log y pedir la QSL por OQRS, y el puesto en el **Most Wanted** de DXCC |

Si el mismo indicativo está en más de una fuente, las fechas, la ruta de QSL y las notas
salen primero de NG3K y después de 425 DX News; Club Log suma su link y, como sus fechas
son QSOs reales, puede estirar las fechas. El mismo indicativo en dos viajes distintos
(ahora y el año que viene) queda como dos entradas. Cada tarjeta de la pestaña
DXpediciones muestra sus fuentes y el puesto en el Most Wanted; la pestaña Estación
tiene el link a cada fuente y al log de Club Log.

**Orden.** **En el aire** pone primero las DXpediciones spoteadas en los últimos 30 minutos, la del spot más reciente arriba, y el resto por Most Wanted; la que pasa 30 minutos sin spots vuelve a su lugar. **Most Wanted** (el que viene elegido) pone primero los países más buscados, según el puesto de Club Log, y al final los que no tienen puesto; **Fecha** ordena por la fecha de inicio anunciada. El navegador recuerda tu elección. 425 DX News a
veces da solo el fin ("till 31/10"); entonces el inicio se lee del texto del boletín
("from 20 September to…", "on 8-11 October", "between 7 and 21 October"). Si el texto
tampoco lo dice, la tarjeta muestra **hasta 10-31** y va primero, porque ya está activa.

**En el aire.** Una DXpedición spoteada (cluster o RBN) en los últimos 30 minutos se
marca con un borde verde y una línea **EN EL AIRE** con la frecuencia (clic para
sintonizar), banda, modo, hace cuánto y quién la spoteó. La pestaña DXpediciones muestra
cuántas hay en el aire (**● 4**) aunque estés mirando la pestaña Estación, el mapa marca
sus rombos, y aparece un aviso en el panel apenas una sale al aire. Pasa con todas las
DXpediciones activas, las necesites o no y estén o no en tu lista de seguimiento;
**solo las que faltan** nunca oculta una que está en el aire.

### ¿Estás en su log?

Cuando abrís una DXpedición activa en la pestaña Estación y su log está en Club Log, el
recuadro de la DXpedición busca tu indicativo en el buscador de log de Club Log: los
casilleros de banda y modo donde estás en su log (✓), los que ya activaron y te faltan (·),
y dos avisos: **en tu log pero no en el de ellos** (un indicativo mal copiado, un QSO que no
anotaron o un día que todavía no subieron: se muestra la fecha de la última subida) y **en
su log pero no en el tuyo** (un QSO que te falta cargar). Solo consulta cuando abrís esa
estación y guarda la respuesta 10 minutos; dice a qué hora consultó. ¿Recién los trabajaste?
**↻ Actualizar** vuelve a consultar Club Log en el momento (tu QSO aparece ahí cuando lo suben). Muchas DXpediciones no usan Club Log: en ese
caso muestra las direcciones web de su anuncio (NG3K, 425 DX News), donde suele estar su
propio buscador de log u OQRS.

### ¿Necesito esta DXpedición?

La columna **Qué te falta** del panel (en la terminal, el texto después de `NEED:`)
dice exactamente qué suma trabajarla, lo más importante primero:

| En el panel (ES)                                       | En la terminal / EN                    | Significado                                |
|--------------------------------------------------------|----------------------------------------|--------------------------------------------|
| `ATNO`, `banda nueva`, `modo nuevo`                    | `ATNO`, `new band`, `new mode`         | Algo que nunca trabajaste                  |
| `17m DIGI nunca trabajado`                             | `17m DIGI never worked`                | Tenés la banda y el modo, pero no juntos   |
| `país no confirmado`, `banda no confirmada`, `modo no confirmado` | `entity not confirmed`, `band not confirmed`, `mode not confirmed` | Trabajado, pero sin LoTW ni QSL en papel (eQSL no cuenta) |
| `ya lo tenés: 20m CW confirmado`                       | `have it: 20m CW confirmed`            | Nada nuevo: trabajarla solo si se quiere   |

El título lo resume: **DXPED FALTA** cuando hay algo nuevo, **DXPED CONFIRMAR** cuando
lo que se gana es una confirmación, **DXPED YA TENGO** cuando no se gana nada.

### Colores y notificaciones

| Prioridad | Color          | Notificación | Alertas                                                       |
|-----------|----------------|--------------|---------------------------------------------------------------|
| 1         | blanco en rojo | sí           | ATNO                                                          |
| 2         | amarillo       | no           | BANDA NUEVA; DXPED FALTA/CONFIRMAR por banda o modo nuevos, o país no confirmado |
| 3         | celeste        | no           | MODO NUEVO, SEGUIMIENTO; DXPED por banda+modo nuevos o banda/modo no confirmados |
| 4         | verde          | no           | BANDA+MODO; DXPED por banda+modo no confirmados               |
| 5         | gris           | no           | INDICATIVO NUEVO, DXPED YA TENGO                              |

La columna de notificación es la configuración por defecto
(`notify_max_priority = 1`): por defecto solo ATNO muestra notificación. Para que también
la muestren las de prioridad 2 o 3, se cambia en **Notificaciones de Windows** en la página de
configuración. Cada alerta también se guarda en
`data/dxagent.db` (los últimos `keep_days` días).

### Cerca tuyo: ¿la estación está llegando a tu zona?

Una señal fuerte en otro continente no sirve de mucho si no llega hasta vos. DX Agent
registra **todos** los spots de la RBN y del cluster (no solo los que generan
alerta) junto con la ubicación de quien los recibió, y en cada alerta muestra la
estación más cercana a vos que la escuchó en esa banda en los últimos 15 minutos:

| Se muestra | Significado |
|------------|-------------|
| **● 732 km · 7 dB** (verde) | Se escucha cerca tuyo: una estación dentro de tu radio "cerca" (1.500 km por defecto) la escuchó, con esa SNR |
| **8.176 km** (gris) | Solo se escucha más lejos; ese es el receptor más cercano |
| **—** | Sin reportes de la RBN ni del cluster en los últimos 15 minutos |
| **~** antes de la distancia | Aproximado: un spotter del cluster ubicado en el centro de su país |

- **Dónde están los receptores:** los skimmers de la RBN salen de la lista de skimmers
  de la propia RBN, con grid locator exacto, descargada una vez por día. Los spotters
  del cluster se ubican en el centro de su país, según `cty.csv`.
- **Tu ubicación:** el grid locator de la configuración. Si está vacío, se toma de
  `MY_GRIDSQUARE` en tu log.
- **Actualización:** la columna se refresca cada 20 segundos, porque los receptores
  cercanos suelen reportar una estación un poco después del primer spot.
- **Filtro:** el botón **Solo cerca tuyo** muestra solo las alertas que se escuchan
  cerca tuyo. Para recibir *alertas* solo de esas, marcar **Avisar solo si se escucha
  cerca tuyo** en la configuración; la estación avisa apenas alguien cerca tuyo la
  escucha.
- **Pestaña Estación:** **Quién lo escucha** lista los receptores más cercanos a vos
  primero, con banda, SNR y antigüedad. **Consultar PSKReporter (FT8/FT4)** le
  pregunta a PSKReporter, que tiene muchos más receptores de FT8 en Sudamérica que
  skimmers la RBN. Sus recepciones se suman a "cerca tuyo", así que el resumen y la
  columna de la tabla también se actualizan. La respuesta se guarda 5 minutos por
  indicativo. PSKReporter pide **como máximo una consulta cada 5 minutos** (para todas
  las estaciones juntas) y bloquea las conexiones que consultan más seguido (HTTP
  429), así que DX Agent respeta ese límite, muestra cuándo se puede volver a
  consultar y, después de un bloqueo, espera al menos 15 minutos antes de reintentar.

Cuántos receptores hay cerca tuyo depende de dónde estés: los skimmers de la RBN
abundan en América del Norte y Europa y escasean en otras zonas, mientras que
PSKReporter suele tener muchos más receptores de FT8.

### Alertas repetidas (tiempo de espera)

La misma alerta no se repite dentro del tiempo de espera (15 minutos por defecto):

- **Alertas de país** (ATNO, NEW BAND, NEW MODE, NEW SLOT): se limitan por
  país + banda + modo. Con una estación de EE. UU. en 30 m CW alcanza; durante los
  15 minutos siguientes no llega una alerta por cada otra estación de EE. UU. en
  30 m CW.
- **Alertas de indicativo** (DXPEDITION, WATCHLIST, NEW CALL): se limitan por
  indicativo + banda + modo, así que la DXpedición vuelve a avisar cuando pasa a otra
  banda o modo.

---

## 6. El archivo de configuración

Todo lo de la página de configuración se guarda en `config.toml`, en la carpeta del
proyecto. También se puede editar a mano; después hay que reiniciar `run`.

| Sección     | Ajuste                      | Por defecto                                       | Qué hace |
|-------------|-----------------------------|---------------------------------------------------|----------|
| (inicio)    | `data_dir`                  | `data` (por defecto)                              | Dónde se guardan las descargas, el historial de alertas y la contraseña de LoTW guardada |
| station     | `callsign`                  | (tu indicativo)                                   | Se usa para conectarse al cluster y a la RBN |
| station     | `log_source`                | `adif`                                            | `adif` = un archivo de tu logger; `lotw` = todos los QSOs que subiste a LoTW (sin archivo ADIF) |
| station     | `adif`                      | `C:/Users/you/AppData/Roaming/Log4OM2/backup/CALL_*_backup.adi` | Ruta o patrón del log. Con un patrón se usa el archivo más nuevo. También acepta una lista de rutas |
| station     | `grid`                      | vacío (= se toma del log)                         | Tu grid locator, para "cerca tuyo" |
| alerts      | `alert_on`                  | `atno, dxpedition, new_band, new_mode, watchlist` | Qué tipos de alerta se activan |
| alerts      | `call_scope`                | `band_mode`                                       | Para DXpediciones y lista de seguimiento: `call` = cualquier QSO cuenta, `band` = por banda, `band_mode` = por banda y modo |
| alerts      | `bands`                     | 160m–6m (sin 60m)                                 | Los spots en otras bandas se ignoran |
| alerts      | `modes`                     | `CW, PHONE, DIGI`                                 | Grupos de modos con alertas |
| alerts      | `watchlist`                 | vacío                                             | Indicativos a buscar siempre; admite comodines, por ejemplo `"VP8*"` |
| alerts      | `ignore`                    | vacío                                             | Indicativos que nunca generan alerta; admite comodines |
| alerts      | `cooldown_minutes`          | `15`                                              | Tiempo mínimo antes de repetir la misma alerta |
| alerts      | `desktop`                   | `true`                                            | Notificaciones de Windows (incluidas en Windows 10/11) |
| alerts      | `notify_max_priority`       | `1`                                               | Notificaciones para: 1 = solo ATNO; 3 = también modo nuevo y lista de seguimiento |
| alerts      | `near_km`                   | `1500`                                            | "Cerca tuyo" = la escuchó una estación a esta distancia en km o menos |
| alerts      | `near_only`                 | `false`                                           | Avisar solo si la estación se escucha cerca tuyo |
| alerts      | `keep_days`                 | `7`                                               | Días de alertas que se guardan en `data/dxagent.db` (0 = ninguna). Con la RBN, unas 25.000 alertas por día, unos 15 MB |
| ui          | `enabled`                   | `true`                                            | Iniciar el panel web con `run` |
| ui          | `port`                      | `8073`                                            | Dirección del panel: `http://localhost:8073` |
| ui          | `open_browser`              | `true`                                            | Abrir el panel automáticamente al iniciar `run` |
| logbook     | `reload_seconds`            | `60`                                              | Cada cuánto revisar si cambió el archivo ADIF |
| logbook     | `live`                      | `true`                                            | Leer también, cada 10 segundos, los QSOs que Log4OM 2 guardó desde su último backup, de su base |
| logbook     | `udp_port`                  | `12065`                                           | QSOs que tu programa de log manda por UDP al loguearlos (N1MM, DXLog.net, Log4OM, WSJT-X; ver "Tus QSOs a medida que los logueás"); 0 = apagado |
| logbook     | `udp_group`                 | `""`                                              | Un grupo multicast al que unirse en ese puerto (el UDP Server de WSJT-X, por ejemplo `239.255.0.1`) |
| dxpeditions | `refresh_hours`             | `6`                                               | Cada cuánto releer las fuentes de DXpediciones |
| dxpeditions | `sources`                   | `["ng3k", "425dxn", "clublog"]`         | Dónde buscar DXpediciones: NG3K ADXO, 425 DX News, Club Log |
| dxpeditions | `show_have`                 | `true`                                            | Mostrar las líneas DXPED HAVE (gris, sin notificación) |
| lotw        | `enabled`                   | `true`                                            | Descargar tus confirmaciones de LoTW |
| lotw        | `username`                  | vacío (= tu indicativo)                           | Usuario de LoTW (vacío = tu indicativo) |
| lotw        | `password_env`              | `LOTW_PASSWORD`                                   | Nombre de una variable de entorno de la que leer la contraseña (opcional, ver LoTW) |
| lotw        | `refresh_hours`             | `12`                                              | Cada cuánto descargar confirmaciones nuevas |
| cluster     | `enabled`                   | `true`                                            | Usar el DX cluster |
| cluster     | `hosts`                     | `w3lpl.net:7373`, `dxspider.co.uk:7300`, `dx.k3lr.com:23` | Servidores de cluster, en orden. Si uno falla o no envía nada durante 5 minutos, se usa el siguiente |
| cluster     | `commands`                  | vacío                                             | Comandos enviados al conectar, por ejemplo `["set/ft8"]` |
| cluster     | `history_minutes`           | `30`                                              | Al conectar, pedirle al cluster sus últimos spots (`sh/dx 100`) y quedarse con los de los últimos N minutos; `0` = no |
| cluster     | `spotter_continents`        | vacío (todos)                                     | Usar solo spots de spotters de estos continentes |
| rbn         | `enabled`                   | `true`                                            | Usar la Reverse Beacon Network |
| rbn         | `cw` / `digital`            | `true` / `true`                                   | Fuente CW+RTTY (puerto 7000) y FT8/FT4 (puerto 7001) |
| rbn         | `spotter_continents`        | `SA, NA`                                          | Avisar solo cuando un skimmer de estos continentes escucha la estación |
| rig         | `target`                    | `none`                                            | Botón Sintonizar: `none`, `log4om`, `n1mm`, `omnirig`, `hamlib`, `flrig` o `dxlab` |
| rig         | `host` / `port`             | `127.0.0.1` / `0`                                 | Dónde escucha ese programa; 0 = su puerto por defecto |
| rig         | `set_mode`                  | `true`                                            | También cambiar el modo (CW, USB/LSB, datos/RTTY) |
| rig         | `omnirig_rig`               | `1`                                               | Radio 1 o 2 de OmniRig |

La contraseña de LoTW nunca se guarda en este archivo (ver más abajo).

### Consejos de ajuste

- **Demasiadas alertas.** En las bandas donde tu log tiene pocos países, NEW BAND
  aparece seguido. Dejar marcadas solo **ATNO**
  y **DXpedición**, o desmarcar las bandas que no se operan.
- **Solo estaciones que probablemente se escuchen.** En **Solo skimmers en**, elegir
  solo tu continente. Es una mejor señal de que hay propagación hacia vos, pero donde
  hay pocos skimmers se van a perder algunos spots.
- **Ocultar DXpediciones que ya se tienen.** Desmarcar **Mostrarlas igual**.
- **Buscar estaciones puntuales.** Agregarlas a la **Lista de seguimiento**, por ejemplo `3Y0K` o
  `VP8*`.
- **Repetir alertas antes o después.** Cambiar **No repetir durante**.

### Tus QSOs a medida que los logueás

DX Agent lee tu archivo de log (el backup ADIF) al arrancar y cada vez que cambia. Un programa
de log que le avisa cada QSO al guardarlo lo mantiene al día entre medio: una estación recién
trabajada deja de alertar enseguida, las alertas que ya están en la lista dicen lo que
necesitan ahora y se actualizan las DXpediciones, las activaciones y los slots del mapa. Los
QSOs se suman al archivo hasta que el siguiente backup los trae.

**Log4OM 2** no necesita nada: DX Agent lee los QSOs posteriores al backup directo de la base
de Log4OM, cada 10 segundos (`logbook.live`).

**Por UDP** (`logbook.udp_port`, 12065 de fábrica, en 127.0.0.1): un mismo puerto recibe el XML
de contactos de N1MM, texto ADIF y los mensajes de WSJT-X. En el programa de log se agrega
`127.0.0.1` puerto `12065` como destino, junto a los que ya tengas:

| Programa | Dónde | Probado |
|----------|-------|---------|
| **N1MM Logger+** | Config → Configure Ports… → **Broadcast Data**: marcar **Contacts** y agregar `127.0.0.1:12065` a las direcciones | su formato documentado |
| **DXLog.net** | Options → Configure network: QSO broadcast `127.0.0.1:12065` (hasta tres, separadas por espacios); Options → Broadcast: QSO, y **Use N1MM QSO format** | su formato (el de N1MM) |
| **Log4OM 2** (sin la base) | Settings → Software integration → Connections → UDP outbound, tipo **ADIF_MESSAGE**, `127.0.0.1` puerto `12065` | formato ADIF |
| **WSJT-X / JTDX** | En general no hace falta: tu programa de log recibe sus QSOs y se los avisa a DX Agent. Si loguean solos: Settings → Reporting → UDP Server. Un grupo multicast (`239.255.0.1`, puerto `2237`) deja que DX Agent (`udp_group = "239.255.0.1"`, `udp_port = 2237`) y un programa que se une a grupos (GridTracker, JTAlert) lo escuchen a la vez; una dirección común llega a un solo programa | el mensaje Logged ADIF de WSJT-X |
| **HRD Logbook** | Tools → Configure → QSO Forwarding → **UDP Send**: `127.0.0.1`, puerto `12065` | sin probar: HRD no dice su formato; funciona si manda ADIF o el XML de N1MM |

Sin UDP (DXKeeper, Logger32, N3FJP, otros): un programa que escribe un archivo ADIF a medida
que logueás, o lo exporta solo, se puede agregar como segundo archivo de log en la
configuración, junto al backup; el `wsjtx_log.adi` de WSJT-X es uno. UDP no confirma la
entrega: un paquete perdido solo espera al siguiente backup.

---

## 7. Integración con LoTW

LoTW se usa de dos maneras.

### Quién sube a LoTW (público, sin usuario)

ARRL publica la fecha de la última subida de unos 236.000 indicativos. DX Agent
descarga esa lista una vez por día y agrega el estado de la estación a cada alerta:

| Se muestra        | Significado |
|-------------------|-------------|
| `LoTW 2026-09-22` | La estación sube a LoTW; esa es su última subida |
| `LoTW announced`  | Todavía no subió nada, pero el listado de la DXpedición dice QSL vía LoTW. Los indicativos nuevos de DXpediciones suelen subir recién después del viaje |
| `no LoTW`         | No usa LoTW ni anunció LoTW: la confirmación va a requerir QSL en papel u OQRS |

### LoTW como log (sin archivo ADIF)

Si no tenés un archivo ADIF (o tu logger no lo exporta), DX Agent puede usar LoTW como
tu log: Configuración → Estación → **Tu log** → **Solo LoTW (sin archivo ADIF)**,
después completar usuario y contraseña en la sección LoTW y hacer clic en **Guardar y
aplicar**.

- Baja todos los QSOs que subiste a LoTW, confirmados o no, a `data/lotw_log.adi`. La
  primera vez le pide a LoTW un año por vez (un solo pedido con un log grande puede
  tener a LoTW ocupado tanto tiempo que se corta la conexión), con una barra de progreso
  en el panel. Si LoTW está ocupado o no responde, reintenta solo cada 10 minutos y
  sigue desde el último año completo. Después solo pide los QSOs subidos desde la última
  descarga, cada vez que revisa LoTW (cada 12 horas por defecto).
- La descarga de confirmaciones funciona igual, en segundo plano: mientras tanto el
  agente ya empieza a vigilar los spots.
- Las confirmaciones salen de esa misma descarga más la descarga normal de
  confirmaciones.
- Tu grid locator para "cerca tuyo" se toma de los QSOs (MY_GRIDSQUARE), salvo que lo
  pongas en Configuración.
- **Límite:** solo cuenta lo que subiste a LoTW. Un QSO que anotaste pero nunca subiste
  aparece como "no trabajado".

### Tus confirmaciones (requiere tu contraseña de LoTW)

DX Agent descarga tus QSL confirmadas de LoTW cada 12 horas y las suma al índice del
log. Así "confirmado" se mantiene al día aunque Log4OM no se haya sincronizado con
LoTW. La primera descarga trae todo y puede tardar unos minutos; las siguientes solo
traen las QSL recibidas desde la última descarga.

**Para activarlo:** Configuración → LoTW → marcar **Descargar mis confirmaciones de
LoTW**, escribir la contraseña del sitio de LoTW, hacer clic en **Probar ingreso** y
luego en **Guardar y aplicar**.

Dónde queda la contraseña:

- Se cifra con la protección propia de Windows para tu cuenta de usuario (DPAPI) y se
  guarda en `data/lotw.secret`. Solo tu cuenta de Windows en esta PC puede
  descifrarla. Nunca se escribe en `config.toml`, así que ese archivo se puede
  compartir sin problema.
- Como alternativa, se puede definir una variable de entorno una vez en PowerShell y
  abrir una terminal nueva: `setx LOTW_PASSWORD "tu-contraseña-de-lotw"`. Si existen
  las dos, gana la variable de entorno, y la página de configuración lo indica.

Qué pasa sin contraseña o con una incorrecta:

- Sin contraseña, DX Agent usa las confirmaciones que ya están en tu archivo ADIF y
  al iniciar muestra `no LoTW password set`.
- Con una contraseña incorrecta, **Probar ingreso** dice
  `LoTW rechazó este usuario o contraseña`; mientras funciona muestra
  `LoTW did not return ADIF - check username / password` y usa el log ADIF de la
  misma manera.

### Qué cuenta como confirmado

Un QSO está confirmado cuando `LOTW_QSL_RCVD` o `QSL_RCVD` vale `Y` o `V` en el log, o
cuando viene de la descarga de LoTW. eQSL no cuenta, igual que en las reglas del DXCC.

---

## 8. Cómo funciona

```text
 DX cluster (telnet, w3lpl.net:7373) ─────┐
 RBN CW     (telnet.reversebeacon.net:7000)┼──> lee la línea "DX de ..."
 RBN FT8/FT4 (telnet.reversebeacon.net:7001)┘          │
                                                       v
                          banda + modo (RBN, comentario o plan de bandas)
                                                       │
                                                       v
                    filtros: balizas, indicativo propio, bandas, modos,
                             ignorados, continente del spotter
                                                       │
                                                       v
                         indicativo ──> país DXCC (cty.csv)
                                                       │
                                                       v
   log ADIF + QSL de LoTW ──> índice trabajado / confirmado ──> comparar ──> motivos
   NG3K/425/Club Log ─────> DXpediciones activas ──────────┘        │
                                                                    v
                                           filtrar por alert_on y tiempo de espera
                                                                    │
                                                                    v
                 panel web + línea en la terminal + notificación + data/dxagent.db
```

### Paso a paso

1. **Fuentes.** Tres conexiones telnet funcionan a la vez: el DX cluster, RBN CW y
   RBN FT8/FT4. Cada una responde al pedido de ingreso con tu indicativo (o lo envía
   a los 10 s si no reconoce el pedido). Si una conexión se corta, o un servidor no
   envía nada durante 5 minutos, pasa al siguiente servidor de la lista; cuando todos
   fallaron, espera cada vez más entre intentos, hasta 5 minutos.
   **Spots recientes:** apenas entra al cluster, el agente envía `sh/dx 100` (DXSpider
   y AR-Cluster responden con sus últimos ~100 spots, más o menos la última media hora)
   y procesa los de los últimos `history_minutes`, del más viejo al más nuevo, para que
   la lista no arranque vacía. Generan alertas con su hora real, nunca muestran
   notificación ni suenan, y un spot ya visto no se repite después de una reconexión.
   La RBN no responde a `sh/dx`, así que arranca vacía.
2. **Lectura.** Cada línea `DX de SPOTTER: FREC INDICATIVO comentario HORA` se
   convierte en un spot. Los sufijos de skimmer como `-#` se quitan del indicativo
   del spotter.
3. **Banda y modo.** La banda sale de la frecuencia. El modo sale de la RBN (informa
   CW, RTTY, FT8, FT4), si no del comentario del cluster (`cw`, `ssb`, `ft8`...), si
   no de lo que la RBN informó para ese indicativo a menos de 2 kHz en los últimos 30
   minutos, y si no de la frecuencia: las frecuencias de FT8/FT4 (14074, 10136...) son
   FT8/FT4, y las partes de CW y de fonía del plan de bandas son CW y SSB. Solo un spot
   en una parte que usan varios modos (la digital, la fonía de 160 y 6 m) se muestra
   con `?`. Los modos se agrupan como en el DXCC:
   **CW**, **PHONE** (SSB, AM, FM) y **DIGI** (FT8, FT4, RTTY, PSK, MFSK...).
4. **Filtros.** Se descartan las balizas, tu propio indicativo, las bandas y modos
   que no se quieren, los indicativos de la lista de ignorados y (si está
   configurado) los spotters fuera de los continentes elegidos.
5. **País.** El indicativo se asocia a una entidad DXCC con el `cty.csv` de AD1C:
   primero indicativos exactos, después el prefijo más largo que coincida. Los
   indicativos portables como `TF/PD9DX` o `W9HT/VP9` usan el prefijo de la
   ubicación; `/P`, `/M` y `/QRP` se ignoran; `/MM` y `/AM` no tienen país.
6. **Tu log.** El archivo ADIF se carga en conjuntos de lo que ya tenés: países,
   país+banda, país+modo, país+banda+modo e indicativos, cada uno como "trabajado" y
   como "confirmado". Para los QSOs del log se usa el campo `DXCC` del ADIF cuando
   existe. El archivo se revisa cada 60 segundos y se vuelve a leer cuando cambia
   (unos 3 segundos para 49.000 QSOs). Las confirmaciones de LoTW se suman igual.
7. **Comparación.** El spot se compara con esos conjuntos para encontrar los motivos
   (ATNO, banda nueva, modo nuevo, combinación nueva, DXpedición, lista de
   seguimiento, indicativo nuevo). Para las DXpediciones también calcula qué sumaría
   trabajarlas, incluidas las confirmaciones.
8. **Alerta.** Se descartan los motivos que no están en `alert_on`, se aplica el
   tiempo de espera y la alerta se muestra en el panel, se imprime y se guarda.

Al guardar la configuración, el agente detiene las fuentes, vuelve a cargar la
configuración, el log y LoTW, y reinicia las fuentes. El panel y las alertas que ya
están en pantalla se mantienen.

### Fuentes de datos

| Fuente | Para qué se usa | Dirección |
|--------|-----------------|-----------|
| DX cluster (W3LPL, con DXSpider UK y K3LR como alternativas) | Spots de operadores | `w3lpl.net:7373`, `dxspider.co.uk:7300`, `dx.k3lr.com:23` |
| Reverse Beacon Network | Spots de skimmers, CW/RTTY y FT8/FT4 | `telnet.reversebeacon.net:7000` y `:7001` |
| Archivos de países de AD1C | Indicativo → país DXCC | https://www.country-files.com/ |
| Calendario ADXO de NG3K | DXpediciones anunciadas | https://www.ng3k.com/Misc/adxo.html |
| Calendario de 425 DX News | DXpediciones anunciadas | https://www.425dxn.org/index.php?op=wcal |
| Club Log | DXpediciones subiendo su log ahora, sus logs y OQRS, Most Wanted de DXCC | https://clublog.org/expeditions.php |
| Actividad de usuarios de LoTW (ARRL) | Quién sube a LoTW | https://lotw.arrl.org/lotw-user-activity.csv |
| Informe de LoTW (ARRL) | Tus confirmaciones | https://lotw.arrl.org/lotwuser/lotwreport.adi |
| Lista de skimmers de la RBN | Dónde está cada skimmer (grid locator) | https://www.reversebeacon.net/nodes/ |
| PSKReporter | Quién recibió una estación en FT8/FT4 (solo a pedido) | https://pskreporter.info/ |
| Spots de POTA y WWFF | Quién está activando un parque ahora (cada 2 minutos) | https://api.pota.app/spot/activator, https://spots.wwff.co/static/spots.json |
| Natural Earth (world-atlas) | Costas y fronteras de los países para el mapa | https://www.naturalearthdata.com/ |
| Mapas DXCC de GridTracker | Entidades DXCC, fronteras solo DXCC y zonas CQ para el mapa | https://gitlab.com/gridtracker.org/gridtracker2 |

---

## 9. Archivos del proyecto y datos descargados

El código está en `dxagent/`, un módulo por tarea. Todo lo descargado va a `data/` y
se puede borrar en cualquier momento; se vuelve a descargar cuando hace falta.

| Archivo                     | Función |
|-----------------------------|---------|
| `config.toml`               | Tu configuración (la escribe la página de configuración; es personal y no se comparte) |
| `config.example.toml`       | La configuración inicial para una instalación nueva |
| `start-dx-agent.bat`        | Doble clic para iniciar DX Agent en Windows, sin ventana |
| `start-dx-agent-console.bat` | Lo mismo, en una ventana que muestra todo lo que va informando |
| `tools/make_package.py`     | Arma el zip para compartir (con Python, programa compilado) y, con `--source`, el zip con el código; nunca tus datos personales |
| `config.toml.bak`           | La configuración anterior, que se guarda en cada cambio |
| `README.md`                 | Resumen breve (en inglés) |
| `DOCUMENTATION.md`          | Esta guía en inglés |
| `DOCUMENTACION.md`          | Esta guía |
| `dxagent/__main__.py`       | Línea de comandos: `run`, `dxpeditions`, `check`, `stats` |
| `dxagent/agent.py`          | Ciclo principal: inicia las fuentes, filtra spots, recarga el log, actualiza DXpediciones y LoTW, aplica la configuración guardada |
| `dxagent/config.py`         | Lee `config.toml` con valores por defecto; lo valida y lo escribe para la página de configuración |
| `dxagent/credentials.py`    | Guarda la contraseña de LoTW cifrada para tu usuario de Windows |
| `dxagent/telnet.py`         | Se conecta al cluster y a la RBN, ingresa y reconecta |
| `dxagent/spots.py`          | Convierte las líneas `DX de ...` en indicativo, frecuencia, banda, modo, SNR, WPM |
| `dxagent/bands.py`          | Banda según la frecuencia, grupos de modos, modo según el plan de bandas |
| `dxagent/cty.py`            | Indicativo → país DXCC, con `cty.csv` |
| `dxagent/adif.py`           | Lee archivos ADIF, incluidos nombres con acentos (UTF-8) |
| `dxagent/logbook.py`        | Arma el índice de trabajados y confirmados a partir del log y de LoTW |
| `dxagent/matcher.py`        | Decide los motivos de alerta y qué sumaría una DXpedición |
| `dxagent/dxpeditions.py`    | Lee y une las listas de DXpediciones de NG3K, 425 DX News y Club Log |
| `dxagent/lotw.py`           | Lista de usuarios de LoTW, prueba de ingreso y descarga de confirmaciones |
| `dxagent/alerts.py`         | Línea en la terminal, colores, notificación, tiempo de espera, historial |
| `dxagent/queries.py`        | Grilla de la estación, lista de DXpediciones y avance en el DXCC, compartidas por los comandos y el panel |
| `dxagent/reach.py`          | "Cerca tuyo": quién escuchó cada estación y a qué distancia de vos; lista de skimmers de la RBN; PSKReporter |
| `dxagent/geo.py`            | Grid locators y distancias |
| `dxagent/onlinelog.py`      | Busca tus QSOs en el log de una DXpedición en Club Log |
| `dxagent/wsjtx.py`          | Manda un indicativo a WSJT-X (mensaje UDP Configure) |
| `dxagent/activities.py`    | Activaciones (POTA, SOTA, IOTA...) leídas del comentario de los spots |
| `dxagent/hunting.py`       | Cumbres, parques, islas... que trabajaste, banda por banda |
| `dxagent/livelog.py`       | Los QSOs que Log4OM guardó desde su último backup, de su base |
| `dxagent/udplog.py`        | QSOs que otros programas mandan por UDP al loguearlos (XML de N1MM / DXLog.net, ADIF, WSJT-X) |
| `dxagent/store.py`         | `data/dxagent.db` (SQLite): los registros propios de DX Agent |
| `dxagent/worldmap.py`       | Tierra, fronteras DXCC y entidades DXCC para el mapa |
| `dxagent/rig.py`            | Sintonizar: se comunica con Log4OM, N1MM, OmniRig, Hamlib, flrig o DXLab |
| `dxagent/web.py`            | Servidor del panel: página, API JSON, eventos en vivo, configuración |
| `dxagent/static/index.html` | El panel y la página de configuración (HTML, CSS y JavaScript en un solo archivo) |
| `dxagent/static/logo.svg`  | El logo de DX Agent (encabezado del panel e ícono de la pestaña) |
| `tests/test_dxagent.py`     | 117 pruebas automáticas (`python -m unittest tests.test_dxagent`) |
| `tests/test_filters.js`     | Los filtros de la lista de alertas, en todas sus combinaciones (los corren las pruebas de arriba si Node.js está instalado) |

| Archivo de datos                              | Origen               | Se actualiza   |
|-----------------------------------------------|----------------------|----------------|
| `data/cty.csv`                                | country-files.com    | cada semana    |
| `data/dxped-ng3k.json`, `dxped-425dxn.json`, `dxped-clublog.json` | NG3K, 425 DX News, Club Log | cada 6 horas |
| `data/clublog-mostwanted.json`                | Most Wanted de Club Log | cada día    |
| `data/lotw-user-activity.csv`                 | ARRL LoTW            | cada día       |
| `data/lotw_qsls.adi` + `data/lotw_state.json` | tu cuenta de LoTW    | cada 12 horas  |
| `data/lotw.secret`                            | página de configuración | al guardar una contraseña |
| `data/rbn-skimmers.json`                      | lista de skimmers de la RBN | cada día |
| `data/sota-summits.csv`                       | lista de cumbres de SOTA | cada mes  |
| `data/world-countries-50m.json`               | Natural Earth (vía world-atlas) | una vez |
| `data/dxcc-shapes.json`, `data/cq-zones.json` | GridTracker                     | una vez |
| `data/world-map.json`                         | armado con los dos de arriba (el mapa terminado) | cuando cambian |
| `data/dxagent.db`                             | DX Agent: alertas (últimos `keep_days` días), dónde estuvo cada activador (últimos 32 días), tus QSOs con ellos | mientras corre |
| `data/dx-agent.log`                           | DX Agent (sin ventana) | mientras corre; menos de 5 MB |

### data/dxagent.db

Los registros propios de DX Agent son un único archivo SQLite (Python ya trae SQLite; no hay
que instalar nada). Todo lo que se baja de afuera sigue siendo el archivo tal como vino: se
reemplaza entero y se lee entero. Las tablas:

| Tabla        | Qué                                                   | Se guarda |
|--------------|-------------------------------------------------------|-----------|
| `alerts`     | cada alerta: hora, indicativo, banda y la alerta en JSON | `alerts.keep_days` días (7) |
| `activators` | dónde estuvo cada activador (indicativo, hora, referencias) | 32 días |
| `hunted`     | tus QSOs con activadores vistos en el aire (indicativo, programa, referencia, banda, hora) | para siempre |

Cualquier programa de SQLite lo abre (DB Browser for SQLite, `sqlite3`), incluso con DX Agent
corriendo. Las versiones anteriores escribían `alerts.jsonl` (y `activations*.jsonl`):
el primer arranque los lee y los renombra `*.migrated`; se pueden borrar cuando quieras. El
`data` de una alerta, por ejemplo (abreviado):

```json
{"utc": "2026-10-13T14:55:04+00:00", "call": "VP2MAW", "freq_khz": 14030.0, "band": "20m",
 "mode": "CW", "mode_group": "CW", "entity": "Montserrat", "dxcc": 96, "reasons": ["dxpedition"],
 "headline": "DXPED HAVE", "need": [], "need_text": "have it: 20m CW confirmed", "lotw": "no LoTW",
 "source": "rbn", "spotter": "WE9V", "snr": 7, "wpm": 27,
 "dxpedition": {"entity": "Montserrat", "end": "2026-10-20", "qsl": "?", "info": "By DK8YY ..."}}
```

---

## 10. Limitaciones conocidas y solución de problemas

### Limitaciones conocidas

- **Un QSO que acabás de guardar, con Log4OM 2:** DX Agent lo lee directo de la base de
  Log4OM en unos 10 segundos, sin esperar al siguiente backup ADIF (`logbook.live`, activado
  de fábrica). Encuentra la base en el `l4ong.status` de Log4OM, la abre solo para leer y
  toma solo los QSOs posteriores al más nuevo del backup. Las alertas que ya están en la lista
  pasan a decir lo que necesitan ahora; también se actualizan las DXpediciones, las
  activaciones y los slots del mapa. El backup sigue siendo el log: se lee entero al arrancar y
  cada vez que cambia. No se lee: Log4OM con MySQL, o un log de Log4OM que no es el de la
  configuración (la base tiene que tener el QSO más nuevo del backup). Las confirmaciones que
  Log4OM recibe de QSOs viejos siguen esperando al backup (o a la descarga de LoTW de DX Agent).
- **Una estación recién trabajada sigue generando alertas, con otros programas de log.** Salvo
  que el programa le avise cada QSO a DX Agent (por UDP: N1MM Logger+, DXLog.net, WSJT-X...) o
  escriba su ADIF a medida que logueás, un QSO nuevo cuenta recién cuando está en el archivo de
  log. Ver "Tus QSOs a medida que los logueás".
- **La descarga de LoTW todavía no se probó con una cuenta real.** La lista pública
  de LoTW funciona; la descarga de confirmaciones y **Probar ingreso** siguen el formato
  de informes de ARRL, pero todavía no se probaron con una contraseña real.
- **Modo deducido de la frecuencia en los spots del cluster.** Cuando ni el
  comentario ni la RBN dicen el modo, sale de la frecuencia (pasando el mouse sobre el
  modo se ve). Una estación fuera de los segmentos habituales (SSB en la parte de CW, el
  FT8 de una DXpedición en su propia frecuencia) puede quedar mal; la parte digital se
  muestra con `?`.
- **Las fechas anunciadas son aproximadas.** Una DXpedición cuenta como activa desde un
  día antes de su fecha de inicio hasta un día después de su fecha de fin.
- **Indicativos portables.** Los indicativos como `TF/PD9DX` se asocian al prefijo
  más corto. En casos raros, como `3D2/R` para Rotuma, el país puede salir mal.
- **El modo con Log4OM va por OmniRig.** El control remoto de Log4OM no puede cambiar el modo
  de la radio, así que DX Agent lo cambia por OmniRig (ver Sintonizar la radio): un Log4OM que
  maneja la radio de otra forma (Hamlib, su propio CAT) recibe solo la frecuencia.
- **60 m está desactivada por defecto.** Elegirla en **Bandas** para vigilarla; para
  60 m no hay deducción de modo por plan de bandas.

### Solución de problemas

| Síntoma | Solución |
|---------|----------|
| Franja amarilla "DX Agent no puede iniciar: No se encontró ningún archivo ADIF" | Abrir la configuración, corregir la ruta del log y usar **Probar** hasta que encuentre el archivo; luego guardar |
| No encuentra `config.toml` | Ejecutar desde la carpeta `dx-agent`, o pasar `-c` con la ruta completa |
| La configuración no se guarda, aparece una lista en rojo junto al botón | Corregir cada punto de la lista; no se guarda nada hasta corregirlos todos |
| Un cambio de configuración empeoró las cosas | `config.toml.bak` tiene la configuración anterior: copiarlo sobre `config.toml` y reiniciar |
| "saved password could not be read" | La contraseña la guardó otro usuario de Windows u otra PC; volver a escribirla en la configuración |
| `[cluster] ... dropped ...; trying ...` | Normal: pasa solo al siguiente cluster de la lista |
| Pocas alertas del cluster | Normal: el cluster envía unos pocos spots por minuto contra varios por segundo de la RBN, y la RBN suele reportar primero la misma estación |
| Algunas alertas no muestran notificación | Revisar **Notificaciones de Windows** en la configuración; DXPED HAVE, NEW SLOT y NEW CALL no muestran notificación por defecto |
| Ninguna notificación | Marcar **Mostrar notificaciones** en la configuración y revisar que las notificaciones de Windows estén activadas (Configuración de Windows → Sistema → Notificaciones, incluida "Windows PowerShell", que es quien las muestra) |
| En la terminal aparecen códigos como `[1;93m` | Usar Windows Terminal o la terminal de VS Code |
| Datos de países incorrectos o viejos | Borrar `data/cty.csv`; se descarga de nuevo en el próximo inicio |
| `[ui] could not start on port 8073` | Otro programa (u otra copia de DX Agent) usa ese puerto; cerrarlo o cambiar el puerto del panel |
| El panel muestra "Se perdió la conexión con el agente" | `run` se detuvo o falló; iniciarlo de nuevo y la página se reconecta sola |
| El panel aparece vacío después de reiniciar | Se llena con los spots del cluster de los últimos 30 minutos apenas se conecta el cluster (salvo que **Spots recientes al conectar** sea 0). Las alertas anteriores están en `data/dxagent.db` (tabla `alerts`) |
| La lista de DXpediciones parece vieja | Borrar los archivos `data/dxped-*.json` |
| Las confirmaciones de LoTW parecen incorrectas | Borrar `data/lotw_qsls.adi` y `data/lotw_state.json` para forzar una descarga completa |
