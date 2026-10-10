# 🌶️ Mamachita · Mapa de territorio

Mapa de las tiendas de la hoja **"Mamachita - Prospectos tiendas Guadalajara y Zapopan"**, con:

- **Pines por estado**: gris = prospecto, dorado = visitada, verde = con producto, naranja = reestock pronto, rojo (late) = reestock vencido, gris oscuro = no interesada.
- **Territorio conquistado** 🚩: un círculo rojo alrededor de cada tienda con producto, porcentaje por zona y niveles ("Antojito" → "Mamachita suprema").
- **Entregas**: frascos por sabor (Tradicional, Pistache/Morita), precio, fecha de reestock, cuántos quedaban, valor dejado y vendidos estimados.
- **Rutas**: "Ruta de reestock" o "Prospectos cerca de mí", ordenadas por cercanía, que se abren en Google Maps.

La app lee la hoja en vivo: cada fila nueva que agregues en `Prospectos` aparece en el mapa al recargar.

---

## 1. Lo mínimo (ya funciona)

Con la hoja compartida como **"Cualquier persona con el enlace: lector"**, el mapa la lee directo.
Las tiendas sin coordenadas se ubican con OpenStreetMap (aproximado, se guarda en tu navegador).

## 2. Conectar el Apps Script (recomendado, 5 min)

Con esto: coordenadas exactas de Google escritas en la hoja, registrar entregas y cambiar estados desde el mapa, y **un correo diario a las 8am** con las tiendas por reestockear.

1. Abre la hoja → **Extensiones → Apps Script**.
2. Borra lo que haya y pega todo el contenido de [`apps-script/Code.gs`](apps-script/Code.gs).
   - Si quieres clave para guardar, escríbela en `const PIN = '...'`.
3. Guarda (💾), elige la función **`configurar`** arriba y dale **Ejecutar**. Acepta los permisos.
   - Esto agrega las columnas `Estado`, `Lat`, `Lng`, crea la pestaña `Entregas` y saca las coordenadas de las 121 tiendas.
4. **Implementar → Nueva implementación → Tipo: Aplicación web**
   - Ejecutar como: **Yo**
   - Quién tiene acceso: **Cualquier usuario**
   - Copia la URL que termina en `/exec`.
5. Pega esa URL en [`config.js`](config.js) → `SCRIPT_URL: 'https://script.google.com/macros/s/.../exec'`.

Desde ahí, cuando escribas una tienda nueva en la hoja, sus coordenadas se llenan solas.

### Columnas

**Prospectos** (las que ya tienes + 3 nuevas): `Prioridad · Zona · Tipo · Nombre · Ubicación · Visitado · Maps · Notas · Estado · Lat · Lng`

- `Estado`: Prospecto, Visitado, Interesado, Activa, No interesado.

**Entregas** (una fila por visita en la que dejas producto):
`Fecha · Tienda · Tradicional · Pistache/Morita · Precio · Reestock · Quedaban · Notas`

- `Tienda` debe escribirse igual que en `Prospectos`.
- `Reestock` vacío = Fecha + 14 días (se cambia en `config.js` → `DIAS_REESTOCK`).
- `Quedaban` = frascos sin vender que encontraste al llegar; con eso se calculan los vendidos.

Para agregar otro sabor: agrega la columna en `Entregas` y una línea en `SABORES` de `config.js`.

## 2b. Buscar tiendas de Google Maps (opcional)

Google Maps tiene muchas más tienditas que OpenStreetMap. Para traerlas hace falta una clave de **Google Places API**:

1. Entra a <https://console.cloud.google.com/> con tu cuenta, crea un proyecto (p. ej. "Mamachita") y **activa la facturación**
   (pide tarjeta, pero el uso de este mapa queda dentro del nivel gratuito mensual de Google).
2. **APIs y servicios → Biblioteca → "Places API (New)" → Habilitar.**
3. **APIs y servicios → Credenciales → Crear credenciales → Clave de API.** Edítala y en "Restricciones de API" elige solo *Places API (New)*.
4. En Apps Script: **⚙️ Configuración del proyecto → Propiedades de la secuencia de comandos → Agregar**
   - Propiedad: `GOOGLE_PLACES_KEY` · Valor: tu clave.
5. Pega el `Code.gs` nuevo, guarda y **actualiza la implementación** (Gestionar implementaciones → ✏️ → Nueva versión).

Con eso:
- En la hoja: menú **🌶️ Mamachita → Buscar tiendas en Google (por zona)** recorre cada zona de tu lista y agrega las tiendas nuevas
  (sin OXXO/7-Eleven/súpers grandes ni repetidas). Si faltan zonas al terminar, vuelve a correrlo y sigue donde se quedó.
- En el mapa: el botón **🔍** busca en Google la zona que estés viendo (si no hay clave, usa OpenStreetMap).

Cada búsqueda por zona son ~6–18 consultas; revisar las ~40 zonas una vez son unas 500 consultas.

## 3. Publicación (mamachita.quisitofood.com)

Este repo se publica con GitHub Pages; el archivo `CNAME` ya apunta al subdominio.

1. **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`** → Save.
2. En el DNS de `quisitofood.com`: registro **CNAME** `mamachita` → `lalo2188.github.io` (sin puerto).
3. Cuando GitHub verifique el dominio, activa **Enforce HTTPS** en la misma pantalla.

Cada cambio que se suba a `main` se publica solo en 1–2 minutos.
