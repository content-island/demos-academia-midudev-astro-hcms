# Fences (Bloques de código en Astro)

Vamos a profundizar en los **componentes de Astro**. Si los observas, se
parecen un poco a Vue: tienes código, HTML y estilos todo en un mismo
archivo.

En la página principal tenemos un *h1* que dice "Astro". Vamos a cambiarlo por un texto definido en una variable.

_./src/pages/index.astro_

``` diff
---
+ const title = "Hola React Alicante";
---

<html lang="es">
    <head>
        <meta charset="utf-8" />
        <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
        <meta name="viewport" content="width=device-width" />
        <meta name="generator" content={Astro.generator} />
        <title>Astro</title>
    </head>
    <body>
-       <h1>Astro</h1>
+    <h1>{title}</h1>
```

¿Qué estamos haciendo aquí?

- Definimos una variable en typescript.
- Usamos esa variable en el HTML, para ello el enlace a datos es igual a React, usamos las llaves `{}` para indicarle que evalue ese código.

Si ejecutamos el proyecto, veremos el nuevo título:

``` bash
npm run dev
```

Y ahora tal vez te preguntes: ¿qué son esas tres rallitas que enjaulan a nuestros código Typescript? Son los **Fences** o **rejas**

Son bloques de código que se ejecutan en el servidor. Si estamos en modo
**SSG (Static Site Generation)**, solo se ejecutan una vez, cuando el
sitio se genera, es decir en el momento del build, si estuvieras en modo **SSR** se ejecutarían en el servidor cada vez que se fuera a servir la página.

Vamos a ver esto de forma más clara: obtendremos un valor aleatorio de una API y lo
mostraremos en la página.

Por ejemplo, existe una API pública que devuelve un dato curioso sobre
perros. Vamos a usarla.

*./src/pages/index.astro*

``` diff
---
+ const res = await fetch("https://dogapi.dog/api/v2/facts");
+ const response = await res.json();
+ const title = response?.data[0]?.attributes?.body ?? "Ooops, ¿la API no funciona?";
- const title = "Hola React Alicante";
---
```

Probemos el resultado en el navegador: deberíamos ver un dato curioso
sobre perros.

Si hacemos un build y miramos los archivos generados en
*./dist/index.html*, veremos el dato del perro ya incluido, porque se
obtuvo en tiempo de compilación.

``` bash
npm run build
```

> Si estamos en modo **SSR (Server-Side Rendering)**, este código se
> ejecutará en cada solicitud del servidor. Nunca se ejecuta en el
> navegador.

¿Pero podemos ejecutar código en el navegador? ¡Por supuesto! Incluso
puedes usar **React**, **Vue** o **Svelte** para eso.

Hagamos un ejemplo simple con JavaScript puro: agregaremos un botón que
obtiene y muestra un dato sobre gatos. El botón se llamará **"Get Cat
Fact"**.

*./src/pages/index.astro*

``` diff
<html lang="es">
    <head>
        <meta charset="utf-8" />
        <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
        <meta name="viewport" content="width=device-width" />
        <meta name="generator" content={Astro.generator} />
        <title>Astro</title>
    </head>
    <body>
        <h1>{title}</h1>
+       <button id="cat-fact-button">Get Cat Fact</button>
+       <h2 id="cat-fact"></h2>
    </body>
</html>

+ <script>
+ const button = document.getElementById("cat-fact-button");
+ const factEl = document.getElementById("cat-fact");
+
+ if (button && factEl) {
+  button.addEventListener("click", async () => {
+    const res = await fetch("https://catfact.ninja/fact");
+    const data = await res.json();
+    factEl.innerText = data.fact;
+  });
+}
+ </script>
```

Si ejecutamos el proyecto, veremos que al hacer clic en el botón se
muestra un dato sobre gatos.

``` bash
npm run dev
```

Ahora podrías preguntarte: ¿cómo depurar esto?

Veamos cómo depurar el código dentro de los **fences** y el código del
**navegador**.

Para depurar **código dentro de fences**:

-   Coloca un punto de interrupción dentro del código del fence.
-   Abre una terminal en modo **JavaScript Debug Terminal** y ejecuta:

``` bash
npm run dev
```

Cuando ejecutes el servidor, se detendrá en el punto de interrupción y
podrás depurar.

**Importante:** en modo de desarrollo local, cada vez que recargues la
página, el código del fence se ejecutará de nuevo. Pero esto solo ocurre
en desarrollo; en producción se ejecuta una sola vez, al compilar.

¿Y cómo depuramos el **código del navegador**? Como siempre: usando las
**DevTools** del navegador.

### Bonus

Puedes extraer este código a un archivo *ts*, pero tendrás que ajustar
un poco el código:

*./src/pages/cat.ts*

``` ts
async function getCatFact() {
  const res = await fetch("https://catfact.ninja/fact");
  const data = await res.json();
  return data.fact;
}

export const setupCatFactButton = () => {
  const button = document.getElementById("cat-fact-button");
  const factEl = document.getElementById("cat-fact");

  if (button && factEl) {
    button.addEventListener("click", async () => {
      const fact = await getCatFact();
      factEl.innerText = fact;
    });
  }
};
```

*./src/pages/index.astro*

``` diff
// (...)

<script>
+ import { setupCatFactButton } from "./cat";
+ setupCatFactButton();
-  const button = document.getElementById("cat-fact-button");
-  const factEl = document.getElementById("cat-fact");
-
-  if (button && factEl) {
-    button.addEventListener("click", async () => {
-      const res = await fetch("https://catfact.ninja/fact");
-      const data = await res.json();
-      factEl.innerText = data.fact;
-    });
-  }
</script>
```
