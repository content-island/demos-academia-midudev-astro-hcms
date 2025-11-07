# Server Streaming

Hasta ahora hemos estado trabajando en modo SSG (Static Site Generation), es decir ejecutabamos el código en tiempo de build y generabamos un sitio estático.

Pero Astro nos ofrece también soporte a Server Side Rendering (es decir que una página se genere cada vez que la pidamos en servidor), y lo que es mejor... módo hibrido, esto es:

- Puedes elegir que unas páginas se generen con SSG.
- Otras que se generen cada vez que se pidan con SSR.

Esto te permite ajustar el rendimiento, para las páginas que no lo necesiten tiras de estático, y para las que necesiten datos en tiempo real de server side rendering.

Y otro tema interesante es que en Server Side Rendering puedes darle prioridad a servir antes ciertas partes de la aplicación (es lo que se llama server streaming).

Vamos a poner toda esta teoría en práctica.

Lo primero que vamos a hacer es crear un nuevo proyecto, esto es una oportunidad estupend para que práctiques, intenta crear un proyecto en blanco por tu cuenta, repasa las guías de los primeros módulos, dale a la pausa e intentalo.

Vamos con la solución.

Para crear el nuevo proyecto en astro ejecutamos:

```bash
npm create astro@latest
```

Queremos que algunas páginas tiren de SSR, para ello tenemos que añadir un adaptador de servidor, vamos a elegir el de nodejs ¿Te acuerdas como hicimos eso? Dale a la pausa e intentalo.

instalamos el adaptador de nodejs para astro:

```bash
npm install @astrojs/node
```

Y lo añadimos al `astro.config.mjs`:

_./astro.config.mjs_

```diff
// @ts-check
import { defineConfig } from 'astro/config';
+ import node from '@astrojs/node';

// https://astro.build/config
export default defineConfig({
+  adapter: node({
+    mode: 'standalone',
+  }),
});
```

Vamos ahora a cargar las imagenes de gatos y perros en la página principal, vamos a meterlo en un fichero de API para que sea más fácil de usar (crearemos esta vez una carpeta `api` debajo de `src`y un fichero que se llame _animal.api.ts_) ¿Te acuerdas como se hacía? Dale a la pausa e intentalo.

_./src/api/animal.api.ts_

```ts
export async function getRandomDogImage(): Promise<string> {
  const imageError =
    "https://www.publicdomainpictures.net/pictures/190000/nahled/sad-dog-1468499671wYW.jpg";

  const res = await fetch("https://dog.ceo/api/breeds/image/random");
  const response: { message?: string } = await res.json();
  return response?.message ?? imageError;
}

export async function getRandomCatImage(): Promise<string> {
  const res = await fetch("https://api.thecatapi.com/v1/images/search");
  const data: { url: string }[] = await res.json();
  return data[0].url;
}
```

Y vamos a mostrarlas

_./src/index.astro_

```astro
---
const dogImage = await getRandomDogImage();
const catImage = await getRandomCatImage();
---

<html>
  <head>
    <meta charset="UTF-8" />
    <title>Random Dog and Cat Images</title>
  </head>
  <body>
    <h1>🐶 Random Dog Image</h1>
    <img
      src={dogImage}
      style="max-width: 400px"
    />

    <h1>🐱 Random Cat Image</h1>
    <img
      src={catImage}
      style="max-width: 400px"
    />
  </body>
</html>
```

Si hacemos un build podemos ver como se genera la página en HTML.

Vamos añadir una línea de código indicándole que la página se genere en servidor cada vez que se haga una petición (SSR).

_./src/index.astro_

```diff
---
+ export const prerender = false;
const dogImage = await getRandomDogImage();
const catImage = await getRandomCatImage();
---
```

Si ahora hacemos un build, ya no tenemos la página HTML, pregenerada, si queremos ver la página tenemos que irnos a:`dist/server/pages/_image.astro.mjs`.

Ahora vamos simular un delay en la carga de la foto de los gatos (es normal, los gatos van a su bola)

```diff
export async function getRandomCatImage(): Promise<string> {
  const res = await fetch("https://api.thecatapi.com/v1/images/search");
  const data: { url: string }[] = await res.json();

+  // ⏳ Add a 5-second delay
+  await new Promise(resolve => setTimeout(resolve, 5000));

  return data[0].url;
}
```

Si recargamos la página ¿Qué pasa? Pues que tarda un rato en cargarse, hasta que el servicio más lento no haya terminado no tenemos nada que hacer, esto puede llevar a problema porque hay veces que tenemos fragmentos de páginas que son más priotarios.

¿Qué podemos hacer? Implementar server streaming.

Vamos a crear un componente perro y otro componente gato, los vamos a crear debajo de `./src/components/dog.astro` y `./src/components/cat.astro`, y los usaremos en index, si te animas a darle a la pausa e implementarlo, adelante :).ñ

_./src/components/dog.astro_

```astro
---
import {getRandomDogImage} from '../api/animal.api';
const dogImage = await getRandomDogImage();
---
<h1>🐶 Random Dog Image</h1>
<img
  src={dogImage}
  style="max-width: 400px"
/>
```

_./src/components/cat.astro_

```astro
---
import { getRandomCatImage} from '../api/animal.api';
const catImage = await getRandomCatImage();
---

<h1>🐱 Random Cat Image</h1>
<img
  src={catImage}
  style="max-width: 400px"
/>
```

Lo usamos en la página.

_./src/index.astro_

```diff
---
export const prerender = false;
+ import Dog from '../components/dog.astro';
+ import Cat from '../components/cat.astro';
- import {getRandomDogImage, getRandomCatImage} from '../api/animal.api';
- const dogImage = await getRandomDogImage();
- const catImage = await getRandomCatImage();
---

<html>
  <head>
    <meta charset="UTF-8" />
    <title>Random Dog and Cat Images</title>
  </head>
  <body>
+    <Dog/>
+    <Cat/>
-    <h1>🐶 Random Dog Image</h1>
-    <img
-      src={dogImage}
-      style="max-width: 400px"
-    />
-
-    <h1>🐱 Random Cat Image</h1>
-    <img
-      src={catImage}
-      style="max-width: 400px"
-    />
  </body>
</html>
```

Si probamos... resulta que ya no bloquea, y es que Astro intenta hacer streaming por defecto, aunque hay casos en los que puede fallar, si queremos asegurarnos, podemos usar la directiva `server:defer`, y además indicarle que si no está el componente listo que muestre un mensaje de cargando...

Vamos ahora decirle a Astro: oye no me bloquees el renderizado del HTML principal esperando a que `Cat` termine de renderizarse en el servidor, envíame la página al navegador lo antes posible y luego inyecta el contenido de `Cat` cuando esté listo.

```diff
    <Dog/>
-    <Cat/>
+    <Cat server:defer>
+			<div slot="fallback">
+  			<span style="color: green; font-size: 2.5rem;">🐱 Loading cat fact...</span>
+			</div>
+		</Cat>
  </body>
```

Vamos ver que pasa:

Y marcamos el de gato como defer.

Ahora si refrescamos la página, tenemos enseguida la foto del perro, y la del gato se sirve despues.

Y podemos añadir un mensaje de cargando.

---

Si estamos en modo SSR, cuando pedimos una página, esta se genera en el servidor y se envía al usuario en un sólo paquete.

¿ Qué pasa si tenemos una páigna rica que carga de diferentes fuentes de datos? Puede que uno de los fragmentos tarde más en llegar que otro, lo que puede hacer que la página tarde en cargarse, y muchas veces lo que nos interesa es que la página se cargue lo más rápido posible.

Una cosa interesante de Astro es que soporte streaming de HTML y de una forma muy sencilla e intituitiva.

## Manos a la obra

Vamos a crear dos páginas, una tirara de streaming de HTML y otra no.

_./src/pages/facts/NoStreaming.astro_

```astro
---
import Layout from "../../layouts/Layout.astro";
---

<Layout>
  <h1>No Streaming</h1>
</Layout>
```

_./src/pages/facts/Streaming.astro_

```astro
---
import Layout from "../../layouts/Layout.astro";

export const prerender = false;
---

<Layout>
  <h1>Streaming</h1>
</Layout>
```

Y en nuestro menú de cabecera vamos a añadir enlaces para poder navegar entre ellas:

_./src/components/Header.astro_

```diff
      <div class="menu__right">
+       <li><a href="/facts/Streaming" class="menu__item">Streaming</a></li>
+       <li><a href="/facts/NoStreaming" class="menu__item">No Streaming</a></li>
        <li><a href="/" class="menu__item">All Post</a></li>
        <li><a href="/about/" class="menu__item">About</a></li>
      </div>

```

Probamos que podemos navegar entre las dos páginas.

Vamos al lio, creamos dos APIs simuladas, una que te de un datos sobre gatitos y otra sobre perritos, la de perritos le vamos a meter un delay de 5 segundos para simular que tarda más en llegar.

_./src/pages/facts/cat-fact.api.ts_

```ts
export async function getCatFact() {
  return "Cats sleep 70% of their lives. 🐱";
}
```

_./src/pages/facts/dog-fact.api.ts_

```ts
export async function getDogFact() {
  await new Promise((resolve) => setTimeout(resolve, 5000)); // Simulate 5s delay
  return "Dogs can learn more than 1000 words. 🐶";
}
```

Vamos a consumir esta API en nuestra página sin streaming:

_./src/pages/facts/NoStreaming.astro_

```diff
---
import Layout from "../../layouts/Layout.astro";
+ import { getCatFact } from "./cat-fact.api";
+ import { getDogFact } from "./dog-fact.api";
+
+ // IMPORTANTE, AQUI DECIMOS QUE VAMOS EN MODO SSR
+ // HIBRIDO
+ export const prerender = false;
+
+ const catFact = await getCatFact();
+ const dogFact = await getDogFact();
---

<Layout>
  <h1>No Streaming</h1>
+  <h2>Cat Fact</h2>
+  <p>{catFact}</p>
+  <h2>Dog Fact</h2>
+  <p>{dogFact}</p>
</Layout>
```

Si probamos a navegar a esta página, veremos que tarda 5 segundos en cargar, ya que estamos esperando a que llegue el dato de los perritos.

Vamos a hacer lo mismo en la página de streaming:

Para ello vamos a romper en componentes la sección que muestra los datos de los gatitos y los perritos:

_./src/pages/facts/components/CatFact.astro_

```astro
---
import { getCatFact } from "../cat-fact.api";

export const prerender = false;

const fact = await getCatFact();
---

<section>
  <h2>Cat Fact</h2>
  <p>{fact}</p>
</section>
```

_./src/pages/facts/components/DogFact.astro_

```astro
---
import { getDogFact } from "../dog-fact.api";

export const prerender = false;

const fact = await getDogFact();
---

<section>
  <h2>Dog Fact</h2>
  <p>{fact}</p>
</section>
```

Y ahora lo usamos tal cual en la página de streaming:

_./src/pages/facts/Streaming.astro_

```diff
---
import Layout from "../../layouts/Layout.astro";
+ import CatFact from "./components/CatFact.astro";
+ import DogFact from "./components/DogFact.astro";

export const prerender = false;
---

<Layout>
  <h2>Streaming</h2>
+ <CatFact />
+ <DogFact />
</Layout>
```

Lo probamos y podemos ver como la página se carga al instante, y los datos de los gatitos llegan antes que los de los perritos.

## BONUS

¿Y si queremos mostrar un loader de la sección que se está cargando?

Aquí vamos a emplear un truco:

- Creamos un componente que tiene dos DIV hermanos.
- El primero DIV se muestra mientras el segundo DVI no sea visibile (esto lo hacemos con un selector de CSS).

Vamos a crear el component fallback:

_./src/components/LoadingFallback.astro_

```astro
---
//https://codehater.blog/articles/zero-js-progressive-loading/
---

<!-- LoadingFallback.astro -->
<div class="contents fallback">
  <slot name="fallback" />
</div>
<slot name="content" />

<style>
  .fallback:has(+ *) {
    display: none;
  }
</style>
```

Este componente está muy chulo, con un selector CSS mira si tiene un hermano y si lo tiene se oculta.

Y vamos a usarlo en nuestra página de streaming:

_./src/pages/facts/Streaming.astro_

```diff
---
import Layout from "../../layouts/Layout.astro";
import CatFact from "./components/CatFact.astro";
import DogFact from "./components/DogFact.astro";
+ import LoadingFallback from "../../components/LoadingFallback.astro";

export const prerender = false;
---

<Layout>
  <h2>Streaming</h2>
  <CatFact />
+  <LoadingFallback>
+   <div slot="fallback">🐶 Loading dog fact...</div>
-    <DogFact />
+   <DogFact slot="content" />
+   </LoadingFallback>
</Layout>
```

Y ya lo tenemos... :)

Otra opción es usar _server islands_, marcamos el componente como _server:defer_ y le añadimos un _fallback_

_./src/pages/facts/components/DogFact.astro_

```diff
<Layout>
  <h2>Streaming</h2>
  <CatFact />
-  <LoadingFallback>
-    <div slot="fallback">🐶 Loading dog fact...</div>
-    <DogFact slot="content" />
-  </LoadingFallback>
+  <DogFact server:defer>
+    <div slot="fallback">🐶 Loading dog fact..</div>
+  </DogFact>
</Layout>
```
