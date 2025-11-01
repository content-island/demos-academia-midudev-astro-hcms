# Estilos

Hasta ahora todo lo que hemos generado tenía una pinta muy fea, vamos a ver como Astro gestiona los estilos.

Si usamos CSS estándar:

- Podemos tener un archivo de CSS global.
- Podemos tener archivos CSS a nivel de componente (que tienen alcance
  limitado al componente).

Astro también admite Tailwind y hay un plugin para ello.

## CSS Global

Vamos a añadir un archivo CSS global en la carpeta `src`, en el que hacemos un reset ligero y definimos algunos estilos básicos globales.

_./src/styles.css_

```css
/* =========================
   Basic global styles
   ========================= */

/* Light reset */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* Global font */
body {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 16px;
  line-height: 1.5;
  color: #333;
  background-color: #f9f9f9;
}

/* Headings */
h1,
h2,
h3,
h4,
h5,
h6 {
  font-weight: bold;
  color: #222;
  margin-bottom: 0.5em;
}

/* Paragraphs */
p {
  margin-bottom: 1em;
}

/* Links */
a {
  color: #007bff;
  text-decoration: none;
}
a:hover {
  text-decoration: underline;
}

/* Buttons */
button {
  cursor: pointer;
  padding: 0.5em 1em;
  border: none;
  border-radius: 6px;
  background-color: #007bff;
  color: white;
  font-size: 1em;
  transition: background-color 0.3s;
}
button:hover {
  background-color: #0056b3;
}

/* General container */
.container {
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
}
```

Y lo usamos en la página principal:

_./src/pages/index.astro_

```diff
---
+ import "../styles.css";
import DogFacts from "../components/DogFact.astro";

const res = await fetch("https://dogapi.dog/api/v2/facts?limit=5");
const response = await res.json();
const data = response?.data ?? [];
const facts: string[] = data.map((item: any) => item.attributes.body);
---
```

Ahora vamos a darle estilo al componente de imagenes de perros:

- Vamos a crear un estilo para un `section` que haga de contenedor de imágenes de perros.

- Vamos a crear una clase `card` para cada imagen de perro, que le dé un borde, un padding y un
  margen, así como el comportamiento `hover`.

- En la imagen vamos a definir un estilo para que las imagenes se muestren lo más homogéneas
  posibles.

Añadimos al final del fichero:

_./src/components/DogPics.astro_

```css
<style>
  section {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 1rem;
    padding: 1rem;
  }

  .card {
    background: #fff;
    border-radius: 1rem;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  .card:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }

  img {
    display: block;
    width: 100%;
    height: 200px;
    object-fit: cover;
  }
</style>
```

Y en el markup:

_./src/components/DogPics.astro_

```diff
---
interface Props {
  urls: string[];
}
const { urls } = Astro.props;
---

- <div>
+ <section aria-label="Dog gallery">
  {urls.map((url) =>
+  <div class="card">
    <img src={url} alt="Dog" />
+  </div>
  )}
+ </section>
- </div>
```

Agreguemos estos estilos al final del componente Astro de los hechos de
perros:

_./src/components/DogFacts.astro_

```astro
<style>
  ul {
    list-style: none;
    padding: 1rem;
    border-radius: 8px;
    background: #f0f4ff;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  }

  li {
    padding: 0.5rem 0;
    border-bottom: 1px solid #ddd;
    color: #333;
    font-size: 1rem;
  }

  li:last-child {
    border-bottom: none;
  }
</style>
```

Si abrimos las herramientas de desarrollo, verás que estos estilos sólo afectan al componente.
