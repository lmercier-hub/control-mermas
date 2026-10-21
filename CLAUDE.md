# Control de Mermas · Grupo Duhau (AED / Economart)

Tablero web que cruza los traslados pendientes de certificar en Finnegans con las descargas que informan
los compradores y arma el archivo IMPORTCERT para importar certificados en Finnegans.

- Sitio: https://lmercier-hub.github.io/control-mermas/ (GitHub Pages, rama `main`, carpeta raíz).
- Todo el tablero está en `index.html`: HTML + CSS + JS en un solo archivo, sin build. SheetJS 0.18.5 se carga desde cdnjs.
- Lo usan dos o más personas de la empresa, cada una con su propio Claude.

## Antes de cambiar algo
1. `git pull` para traer la última versión (otra persona pudo haber subido cambios).
2. Hacer el cambio en `index.html`.
3. Probarlo abriendo la página con un servidor local (por ejemplo `python -m http.server`) y cargando Excels reales.
4. `git commit` con un mensaje en castellano que diga qué cambió, y `git push`. El sitio se actualiza en 1–2 minutos.
5. No borrar ni reescribir el historial (nada de `push --force`).

## Reglas del negocio (confirmadas con el usuario; no cambiarlas sin que lo pida)
- **Pantalla:** kg sin separador de miles, humedad con coma decimal (15,1), fechas d/m/aa (6/6/26).
- **Humedad:** siempre redondeada HACIA ARRIBA a 1 decimal (14,22 → 14,3).
- **Merma de humedad:** Kg recibidos × % de la tabla "% de humedad" embebida en `TABLA_HUMEDAD`. Esa tabla ya incluye el manipuleo: +0,25 soja/maíz/sorgo/maní, +0,20 girasol/cebadas, +0,10 trigo. Es la misma cuenta que hace Finnegans.
- **MERMAOTROS:** Kg recibidos − merma por tabla − kg finales que informa el comprador. Así Finnegans termina en los mismos kg finales del comprador.
  - Si la merma del comprador difiere de la tabla en ±1 kg, se toma la del comprador (redondeo).
  - Si difiere más, la carta se marca para revisar.
- **Cruce:** se cruza por CTG: 11 dígitos, empieza con 101 o 102. La carta de porte siempre va como `00001-00011485` (5 + 8 dígitos). Un pendiente repetido en dos listados de Finnegans se cuenta una sola vez.
- **IMPORTCERT:** 16 columnas, hoja `IMPORTCERT`.
  - CERTIFICADO: número 1..n.
  - CARTAPORTE: texto; se toma la de Finnegans.
  - CTG: número.
  - FECHA: fecha con formato `d/m/yy;@`.
  - PRODUCTO: código.
  - CANTIDADRECIBIDA: kg recibidos.
  - FACTOR y GRADO: 1.
  - HUMEDAD: número, solo si hay merma de humedad.
  - MERMAOTROS: kg.
  - DIMENSION: `DIMCTC`.
  - DIMENSIONVALOR: según el grano.
- **Códigos de grano:**

  | Grano | Código | DIMENSIONVALOR |
  |---|---|---|
  | Maní | 769 | COMMANI |
  | Soja | 31 | COMSOJA |
  | Sorgo | 32 | COMSORGO |
  | Maíz | 68 | COMMAIZ |
  | Girasol | 71 | COMGIRASOL |
  | Trigo pan | 72 | COMTRIGOPAN |
  | Cebada cervecera | 1178 | COMCEBADACERVECERA |
  | Cebada forrajera | 1934 | COMCEBADAFORRAJERA |

- Pellets, urea y fertilizantes no son grano y se descartan. AGD se certifica a mano (no tiene perfil).

## Archivos que reconoce (`PERFILES` en el JS)
- **Finnegans (cuadro 1):** el listado recortado con "Carta de porte" + "Ctg", o el "Reporte de Control de Entregas" completo. En el reporte completo, pendiente = "Certificada" vacía.
- **Compradores (cuadro 2):**
  - **Cargill:** Peso Neto a Certificar, Humedad%, Humedad (kg).
  - **Cargill con hojas Descargas + Calidad.**
  - **Williams Entregas:** dos columnas "Neto Final"; la primera es lo recibido y la última el final.
  - **Entregas del Sur:** C/ Porte, Hdad, Merm.
  - **ACA:** Kg.Netos, %Hum., Kg.Netos S/M. No trae grano: se le agrega una columna "Grano" o se elige en la ficha del archivo.
  - **La Costa Cereales:** ya viene en formato IMPORTCERT.
- **Excel que en realidad son tablas web (HTML guardado como .xls):** por ejemplo el IMPORTCERT que baja La Costa Cereales. Los lee `leerLibro()` como texto, porque el lector de Excel perdía la fila de encabezado (viene sin `<tr>`) y daba vuelta las fechas d/m/aaaa.
- **Para sumar un comprador nuevo:** agregar un perfil en `PERFILES` con `test` (reconoce el encabezado) y `cols` (dónde está cada dato). Después comparar los totales contra un cálculo aparte antes de subirlo.

## Pestaña "Control de facturas"
Es independiente del IMPORTCERT y tiene sus propios datos (`fact`, guardados en localStorage con la clave `LSF`).
- **Cuadro 1:** Reporte de Control de Entregas de Finnegans con los traslados certificados. Columnas: Ctg, Carta de porte, Cantidad origen (kg origen), Recibida (kg recibidos), Certificada (kg finales), Certificado.
- **Cuadro 2:** detalle que manda el comprador como justificativo de la factura. Formato La Costa Cereales (LCC):
  - `x_ctg`: CTG;
  - `neto_destino`: kg de la factura;
  - carta de porte = `NROSUC`-`NROORD` (5 + 8 dígitos);
  - `nombreMe`: producto.
- **Cruce por CTG:** `neto_destino` de la factura contra "Recibida" de Finnegans; tiene que dar exacto.
- **Estados:** Coincide / No coincide / No está en Finnegans / Sin kg para comparar.
- **Reporte:** lista las cartas que no coinciden con CP, CTG, kg de factura, kg recibidos, diferencia, kg origen, kg finales y certificado, y se descarga en Excel.
- **Otros formatos de detalle:** agregar el nombre de su columna de kg en `FAC.neto` (y la de CTG en `FAC.ctg` si hace falta).

## Pestaña "Acondicionamiento"
Es para los camiones que van a acondicionarse por calidad baja. Se carga la carta de porte (CPE de ARCA) y el tablero muestra cada dato listo para copiar y pegar en Finnegans: se toca el dato y queda copiado.
- **Datos que saca:**
  - cabecera: fecha, carta de porte (`00003-00008409`) y CTG;
  - los 16 intervinientes de la sección A, cada uno con CUIT y razón social;
  - grano, campaña y kg (neto, bruto y tara);
  - procedencia y destino: N° de planta, localidad y provincia;
  - km a recorrer y dominios.
- **PDF de ARCA:** trae el texto adentro. Se lee con pdf.js 3.11.174 (cdnjs) y sale exacto.
- **Foto (JPG o PNG) o PDF escaneado:** se lee con OCR (tesseract.js 5.1.1 de jsdelivr, en castellano).
  - Después cada número se vuelve a leer recortado y agrandado con el modelo en inglés; si las dos lecturas no coinciden, el dato queda marcado en amarillo para revisar.
  - Si bruto − tara no da el neto, se prueba con la segunda lectura y, si cierra la cuenta, se corrige solo.
  - El CTG se toma del código QR (jsQR), que trae solo el CTG.
  - Los CUIT se controlan con el dígito verificador.
  - Si un CUIT ya apareció en una carta leída de PDF (o corregida a mano), se usa esa razón social.
  - Con fotos de baja resolución el OCR confunde dígitos (4/9, 6/8). Por eso siempre se avisa que hay que comparar con la carta, y está el botón "Corregir datos".
- **Rótulos deformados por el OCR:** los de la sección A se asignan por parecido (distancia de edición, `parecido()`), no por texto exacto.
- **Cómo se guarda:** cada carta es un documento de `archivos` con tipo `cpe`, sin `partes`:
  - los datos van en `meta.cpe`;
  - también se guardan `meta.fuente` (`pdf` | `ocr`), `meta.dudosos` (lista de `{k, alt}`: el campo y la segunda lectura), `meta.avisos`, `meta.cargada` ("Ya la cargué en Finnegans") y `meta.revisada` (corregida a mano).
  - La misma carta (mismo CTG) cargada de nuevo reemplaza a la anterior.
  - El archivo original no se sube: la foto solo la ve quien la cargó, mientras no recargue la página.
- **Excel:** "Descargar Excel" baja una fila por carta, con las columnas de intervinientes que tengan datos.
- **Fechas:** en esta pestaña van completas (`07/10/2026`), como en la carta, porque se pegan en Finnegans.

## Pestaña "Pendiente acondicionadora"
Revisa que los CTG que informa cada entregador estén en el reporte de traslados de la empresa.
- **Cuatro cuadros de carga** (uno por archivo): reporte de traslados de la empresa, La Costa Cereales, Entregas del Sur y Williams Entregas.
- **Columna de CTG:** la que se llama "Ctg" (en las primeras 15 filas); si no hay, la que tiene más números de 11 o 12 dígitos (`paCol()`).
- **Normalización:** un CTG de 12 dígitos que empieza con 0 se pasa a 11 (`paNorm()`). De los entregadores solo se revisan los de 11 dígitos que empiezan con 102; los repetidos se cuentan una vez y se marcan ×n.
- **Resultado:** cuántos están y cuántos faltan en el reporte, barras por entregador y tabla con filtros. "Copiar para Excel" y "Copiar solo CTG" copian lo filtrado.
- **No se guarda en Firebase ni en localStorage:** vive en memoria (`pa`). Al recargar la página hay que volver a cargar los archivos.

## Pestaña "Facturas de acondicionamiento"
Facturas de las acondicionadoras (secado, paritaria, zarandeo, fumigación, mezcla, bactericida), controladas contra su detalle. Reemplaza al "Panel de Acondicionamiento" que estaba en un artifact de Claude.
- **Secciones:** Resumen, Facturas, CTG duplicados, Cargar y Descuentos pactados.
- **Lector propio, sin Claude:** los PDF se leen con pdf.js (`afPaginas()` → renglones de texto por hoja). `AF_PROVS` tiene una entrada por acondicionadora: `test` reconoce al emisor, `esFactura`/`esTicket` reconocen la hoja y `factura()`/`ticket()` la leen. `afArmar()` junta factura y tickets por CTG y arma los controles (`checks`).
  - **Don Fernando:** AgroCereales Don Fernando (factura con un renglón por CTG + "Detalle" con un ticket de entrada por hoja: kg bruto/tara/neto, humedad y costos por concepto).
  - **Florencia Cereales:** la factura trae un renglón por concepto y CTG ("Zaranda | CTG N° ..."), así que el desglose sale de la factura. El detalle es un "Ticket De Balanza" por CTG (un PDF por camión) impreso como dibujo: los números no son texto y se leen con OCR (`afOcrHoja()`, Tesseract modo 4, escala 3). Del ticket salen kg bruto/tara/descargados, humedad, el "Imponible" (total de servicios) y los kg de servicio. Se controla Imponible = suma de la factura por CTG, zarandeo/fumigada/secada uno por uno, bruto − tara = descargados, y que los kg de servicio de los tickets den los "Kg Descontados por servicios" de la factura. Florencia expresa los servicios también en kilos de grano, pero se pagan en pesos (confirmado con Luis): es solo un control. Los kg de servicio = (servicios + IVA) ÷ "Precio" $/TN del ticket. Se controla por ticket (tolerancia 3 kg) y que kg × precio de todos los tickets dé el total de la factura; el veredicto lo muestra como "equivalente en grano, solo informativo".
  - Cualquier hoja con menos de 40 dígitos de texto se lee con OCR (`afPaginas()`), así que también sirve para PDF escaneados.
  - **Sin lector todavía:** Vittori, SEAC, Cereales Bahía y Agro World. Se reconocen para agrupar, pero sus PDF dan aviso. Para sumar una: pedir 1–2 facturas y detalles reales, completar su entrada en `AF_PROVS` y probar que todos los controles cierren.
  - Montos con `afNum()` (1.859.988,00); `toNum()` no sirve para montos con decimales. CTG siempre de 11 dígitos (`afCtg()`).
- **Controles de cada factura:** cada CTG tiene su ticket y el ticket suma lo mismo que el renglón de la factura; los renglones suman el subtotal; descuento = % × subtotal; subtotal − descuento = neto; IVA 21 % del neto; neto + IVA = total. Tolerancia de $ 1.
- **Cuentas (`afCalc()`), confirmadas con Luis:**
  - Siempre en pesos y sin IVA, con el descuento aplicado. USD × TC de la factura; sin TC no se suma (aviso).
  - El descuento se reparte en cada CTG y concepto en proporción (factor = neto ÷ subtotal).
  - Toneladas = kg que entraron a la acondicionadora (Neto del ticket = bruto − tara).
  - $/TN neto = neto ÷ toneladas; $/TN sin descuento; descuento ponderado; paritaria, secado y otros por TN.
  - Secado por punto = secado neto ÷ Σ(TN × (humedad − base)). La base por grano la carga el usuario en "Descuentos pactados"; si no está, no se calcula (no se inventa).
  - Si falta un dato (toneladas, TC, neto), la factura queda afuera de esa cuenta y se avisa. Sin neto se estima total ÷ 1,21 y se avisa.
- **Circuito de pagos:** al cargar factura + detalle se muestra el veredicto (`afVeredicto()`): "Está bien, se puede mandar a pagos" o "No la mandes a pagos" con lo que no cierra (`afProblemas()`: controles en falso y descuento distinto al pactado). Estados en la lista: Lista para pagos / No coincide / Incompleta / Descuento distinto al pactado / Mandada a pagos. El OK final para pagos se da en el sistema de la empresa; acá "Ya la mandé a pagos" solo deja la marca y guarda `meta.f.pagos`, `pagosPor` y `pagosEn`.
- **Descuento pactado:** se carga por acondicionadora. Si una factura trae otro descuento, queda "Descuento distinto al pactado". Ojo: Vittori trae el descuento metido en el precio de cada línea.
- **Importar del panel anterior:** un .json con `facturas` en el formato viejo (`afDesdePanel()`). Ese archivo NO va al repositorio (el repo es público y tiene montos y CTG).
- **Cómo se guarda:** cada factura es un documento de `archivos` tipo `acf` (datos en `meta.f`, sin `partes`); la misma acondicionadora + número reemplaza a la anterior. Descuentos pactados y humedades base: un documento tipo `acfcfg` (`meta.pactos`, `meta.bases`).
- **No meter facturas, montos ni CTG reales en el código:** el repositorio es público.

## Datos compartidos (Firebase)
- **Proyecto:** Firebase `control-mermas-b902e`, consola de Luis. La configuración web está en `FB_CONFIG` dentro de `index.html`; no es secreta.
- **Ingreso:** con Google. Solo entran los mails que figuran en las reglas de Firestore. Hoy son `lmercier@admin.com.ar`, `jaquerreta@admin.com.ar` y `bpugliese@admin.com.ar`; copia de las reglas en `firestore.rules`.
- **Para sumar a alguien:** agregar su mail en las reglas, en Firebase → Firestore → Reglas → Publicar, y actualizar `firestore.rules`.
- **Colecciones:**
  - `archivos`: un documento por Excel cargado, con nombre, tipo (`fin` | `buy` | `cert` | `fac`), meta, cantidad de partes, `subidoPor`, `subidoEn` y `orden`. También uno por carta de porte de acondicionamiento (tipo `cpe`, sin partes), uno por factura de acondicionamiento (tipo `acf`) y uno con los descuentos pactados (tipo `acfcfg`).
  - `partes`: las filas de cada Excel, de a 400, con `archivoId` e `idx`. Solo se guardan los campos de `CAMPOS`; lo demás se recalcula en `rebuild()`.
- **Sincronización:** cada navegador escucha las dos colecciones y rearma el tablero con `rebuild()`. No se guarda nada en localStorage salvo la pestaña elegida.
- **Al cargar archivos:**
  - un listado nuevo de Finnegans borra los anteriores;
  - un archivo con el mismo nombre y tipo reemplaza al anterior.
- **Quitar y vaciar:** quitar un archivo o "Vaciar todo" borra los documentos de la base para todos.

## Diseño
- Franja verde AED (`--brand-deep`) con el logo Grupo Duhau embebido como webp, y seis pestañas: Resumen, Cargar archivos, Control de facturas, Acondicionamiento, Pendiente acondicionadora y Facturas de acondicionamiento.
- Las pestañas "Cartas de porte" y "Pendientes Finnegans" se sacaron porque no se usaban (pedido de Luis, 20/10/2026). Sus vistas (`view-cartas`, `view-pendientes`) siguen en el HTML, ocultas, porque el código las sigue llenando; para volver a mostrarlas hay que reponer el botón en `.tabs` y sumarlas a `TABS`. En el Resumen, las tarjetas de estado y la lista "Para revisar" ya no son enlaces.
- **Resumen:** seis tarjetas arriba: Listas para el IMPORTCERT, Kg recibidos, Merma de humedad, Otras mermas, Mermas totales y Kg finales. "Mermas totales" = (merma de humedad + otras mermas) ÷ kg recibidos, en %; respeta los filtros de arriba.
- Colores de gráficos, validados para daltonismo:
  - claro: humedad `#2A78D6`, otras mermas `#B8860B`, kg `#1F7A37`;
  - oscuro: `#3987E5`, `#B58714`, `#3A9D52`.
- Cada gráfico tiene su tabla en "Ver datos". Los textos de la interfaz van en castellano rioplatense.
