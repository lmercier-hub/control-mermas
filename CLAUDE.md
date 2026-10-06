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

## Datos compartidos (Firebase)
- **Proyecto:** Firebase `control-mermas-b902e`, consola de Luis. La configuración web está en `FB_CONFIG` dentro de `index.html`; no es secreta.
- **Ingreso:** con Google. Solo entran los mails que figuran en las reglas de Firestore. Hoy son `lmercier@admin.com.ar` y `jaquerreta@admin.com.ar`; copia de las reglas en `firestore.rules`.
- **Para sumar a alguien:** agregar su mail en las reglas, en Firebase → Firestore → Reglas → Publicar, y actualizar `firestore.rules`.
- **Colecciones:**
  - `archivos`: un documento por Excel cargado, con nombre, tipo (`fin` | `buy` | `cert` | `fac`), meta, cantidad de partes, `subidoPor`, `subidoEn` y `orden`.
  - `partes`: las filas de cada Excel, de a 400, con `archivoId` e `idx`. Solo se guardan los campos de `CAMPOS`; lo demás se recalcula en `rebuild()`.
- **Sincronización:** cada navegador escucha las dos colecciones y rearma el tablero con `rebuild()`. No se guarda nada en localStorage salvo la pestaña elegida.
- **Al cargar archivos:**
  - un listado nuevo de Finnegans borra los anteriores;
  - un archivo con el mismo nombre y tipo reemplaza al anterior.
- **Quitar y vaciar:** quitar un archivo o "Vaciar todo" borra los documentos de la base para todos.

## Diseño
- Franja verde AED (`--brand-deep`) con el logo Grupo Duhau embebido como webp, y cuatro pestañas: Resumen, Cargar archivos, Cartas de porte y Pendientes Finnegans.
- Colores de gráficos, validados para daltonismo:
  - claro: humedad `#2A78D6`, otras mermas `#B8860B`, kg `#1F7A37`;
  - oscuro: `#3987E5`, `#B58714`, `#3A9D52`.
- Cada gráfico tiene su tabla en "Ver datos". Los textos de la interfaz van en castellano rioplatense.
