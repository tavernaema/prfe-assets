# FORMATO DIARIO v2 — cambios a aplicar en `pf_html.py`

Rediseño del folleto A4. **No toca el Excel, ni `CONFIG`, ni `pf_data.py`, ni `pf_wsp.py`.**
Los archivos de referencia son `pf_estilos_v2.css` (CSS completo) y `MUESTRARIO_grids_v2.html`
(los 8 grids renderizados con datos reales del 14/08).

---

## 1. Qué cambia visualmente

| | v1 | v2 |
|---|---|---|
| Banner | degradé 135° del color de sección | barra negra `#14150F` + puntera de 7mm en el color de sección |
| Color de sección (hex de CONFIG) | banner + footer | **puntera del banner + borde de la card + color del sabor** |
| Footer de precio | degradé de sección, número blanco (o amarillo si pallet) | **etiqueta amarilla `#FFD23F` fija, número en tinta `#14150F`** |
| Precio secundario (bulto con pallet) | blanco chico | **mismo tamaño** que el principal, `opacity:.72` |
| Fondo de página | `#fff` | `#F7F5F1` (la card queda blanca y se despega) |
| Bordes | `border-radius:2.5mm` + sombra | rectos, sin sombra, borde `.4mm` negro `#14150F` |
| Grids | 1,2,3,4,41,7,9 | los mismos **+ `grid6`** (lista densa) |

El amarillo deja de ser el color del número y pasa a ser el fondo de la etiqueta. La regla
"amarillo = precio" se mantiene, invertida.

---

## 2. Cambios en el markup

Solo se agrega **un div** (`.pb-tab`) y **una clase condicional** (`tag-right`). El resto de los nombres de clase
(`.page`, `.page-banner`, `.pb-logo`, `.pb-title`, `.pb-date`, `.page-body`, `.card`,
`.card-img`, `.rib`, `.no-img`, `.card-main`, `.card-body`, `.card-name`, `.card-pres`,
`.card-sab`, `.card-foot`, `.price-pair`, `.price-single`, `.pblk`, `.pval`, `.plbl`,
`.span2`, `.big3`) queda **igual que en v1**.

### 2.1 Banner

```html
<div class="page-banner">
  <div class="pb-tab"></div>          <!-- NUEVO: puntera amarilla -->
  <div class="pb-in">                 <!-- NUEVO: wrapper del contenido -->
    <img class="pb-logo" src="...">
    <div class="pb-title">INGRESOS DEL DIA</div>
    <div class="pb-date">14/08</div>
  </div>
</div>
```

El `style="--sec:#RRGGBB"` sigue en `.page` (ya no hacen falta `--sec-l` / `--sec-d`:
**se pueden borrar los 3 tonos derivados en `pf_data.py`**, v2 usa un solo hex plano).

### 2.2 Card

No hay spine ni borde de color: la card es blanca con borde negro. El color de sección solo
aparece en la puntera del banner y en el texto de sabores.

```html
<div class="card">                    <!-- + " tag-right" según 2.3 -->
  <div class="card-img">
    <img ...><div class="no-img">MAR</div>
  </div>
  <div class="card-main">             <!-- solo cuando la etiqueta va ABAJO -->
    <div class="card-body">
      <div class="rib">NUEVO PRODUCTO</div>  <!-- CAMBIA: ahora DEBAJO de la imagen,
                                                  primer hijo de .card-body -->
      ...
    </div>
    <div class="card-foot">...</div>
  </div>
</div>
```

### 2.3 Etiqueta abajo vs. a la derecha

Dos variantes, se eligen con la clase `tag-right` en la `.card`:

- **Etiqueta abajo** (default): `.card > .card-img + .card-main( .card-body + .card-foot )`
- **Etiqueta a la derecha**: `.card.tag-right > .card-img + .card-body + .card-foot`
  (sin `.card-main` — el foot es hermano directo)

Regla por grid:

| grid | etiqueta | por qué |
|---|---|---|
| 1 | abajo | card full page — **reservado para ofertas** |
| 2 (sección `pallet=NO`) | **derecha** | precio único, columna amarilla dominante |
| 2 (sección `pallet=SI`) | **abajo** | dos precios lado a lado (pallet + bulto) |
| 3 | derecha | |
| 4 | abajo | card vertical |
| 41 | derecha | |
| 6 | derecha | precio resaltado, sin fondo de bloque |
| 7 / 9 | abajo | card chica vertical |

La sección pallet ya marca `<div class="page pallet">` en v1 — el CSS usa `.page.pallet .grid2`.

---

## 3. `grid6` — nuevo layout

6 filas full-width por página. Layout tipo listado: miniatura 30mm, nombre + presentación +
sabor, precio a la derecha resaltado en amarillo (sin bloque de fondo).
Lleva un encabezado de tabla antes de las cards:

```html
<div class="page-body grid6">
  <div class="thead"><div class="h1">Producto</div><div class="h3">Precio</div></div>
  <div class="card tag-right">...</div>   <!-- ×6 -->
</div>
```

Agregar la fila a la tabla de grids del README:

| grid | cards/pág | Layout |
|------|-----------|--------|
| **6** | 6 | 6 filas full-width tipo listado. Precio resaltado en amarillo a la derecha. Para colas largas de productos. |

En el 14/08 la sección con 33 items pasa de **11 páginas (grid3) a 6 (grid6)**;
el folleto completo baja de **29 a 24 páginas**.

---

## 4. Precio: qué emite `pf_html.py`

Sin cambios de estructura. Lo único: el bloque de precio secundario (bulto cuando hay pallet)
lleva la clase `sec`:

```html
<div class="card-foot">
  <div class="price-pair">
    <div class="pblk"><div class="pval">$669</div><div class="plbl">POR PALLET</div></div>
    <div class="pblk sec"><div class="pval">$679</div><div class="plbl">POR BULTO</div></div>
  </div>
</div>
```

Con un solo precio: `<div class="price-single">` y un solo `.pblk` sin `sec`.
El label custom por IMG_KEY (`DEFAULT_PP_LBL`, "llevando N bultos") sigue igual.

---

## 5. Orden de aplicación sugerido

1. Reemplazar el bloque `<style>` de `pf_html.py` por `pf_estilos_v2.css` completo.
2. Agregar `.pb-tab` + `.pb-in` en la función que arma el banner.
3. Meter la lógica de `tag-right` (tabla de 2.3) donde se decide el markup de la card.
4. Agregar `grid6` al dispatcher de grids (6 cards por página + `thead`).
5. Correr `py gen.py 14-08` y comparar contra `INGRESOS_14-08_v2.html`.

### Verificación

Chequear que ningún `.page-body` desborde:

```js
[...document.querySelectorAll('.page-body')]
  .map((b,i) => b.scrollHeight > b.clientHeight + 2 ? i : null)
  .filter(x => x !== null)   // tiene que dar []
```

---

## 6. Pendiente / no resuelto

- `--sec-l` y `--sec-d` quedan sin uso; si `pf_data.py` los deriva, se pueden borrar
  (no rompen nada si quedan).
- El `.pval` puede reportar overflow horizontal en el chequeo de abajo si Barlow Condensed
  no cargó (la fuente de fallback es ~25% más ancha). Con la tipografía real entra.
- Cards agrupadas con muchos sabores en `grid7`/`grid9`: el sabor está clampeado a 1 línea
  y se corta con `…`. Si molesta, subir a 2 líneas achicando `.card-img` a 37mm.
