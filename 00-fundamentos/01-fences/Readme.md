# 🪐 Fences

Partimos de la demo anterior **00-boilerplate**. Simplemente copia ese proyecto en una carpeta limpia y ejecuta `npm install` y después `npm run dev`.

Vamos a adentrarnos en los componentes de Astro. Si te fijas, se parecen un poco a los de Vue: tienes código, HTML y estilos, todo en el mismo archivo.

Probemos algo: vamos a cambiar el _h1_ de la página principal por un texto definido en una variable.

_./src/pages/index.astro_

```diff
---
+ const title = "¡¡ Hola MiduDev !!";
---

<html lang="en">
	<head>
		<meta charset="utf-8" />
		<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
		<meta name="viewport" content="width=device-width" />
		<meta name="generator" content={Astro.generator} />
		<title>Astro</title>

	</head>
	<body>
-		<h1>Astro</h1>
+    <h1>{title}</h1>
```

> El binding funciona exactamente igual que en React: usamos llaves `{}` para indicar una variable.

Si lo ejecutamos, veremos el nuevo título.

```bash
npm run dev
```

Y ahora quizá te preguntes: ¿qué son los _Fences_?

Son bloques de código que se ejecutan en el servidor. Si estamos en modo **SSG**, solo se ejecutan una vez: cuando se genera el sitio.

Vamos a verlo más claro: vamos a obtener un valor aleatorio desde una API y mostrarlo en la página.

Por ejemplo, existe una API pública que devuelve datos fotitos de perros

_./src/pages/index.astro_

```diff
---
const title = "Hola Open Commit 2025";
+ const imageError = "https://www.publicdomainpictures.net/pictures/190000/nahled/sad-dog-1468499671wYW.jpg";
+ const res = await fetch("https://dog.ceo/api/breeds/image/random");
+ const response = await res.json();
+ const dogImageUrl = response?.message ?? imageError;
---
```

Y actualizamos para mostrar una imagen:

_./src/pages/index.astro_

```diff
	<body>
     <h1>{title}</h1>
+    <img src={dogImageUrl} alt="Random Dog" style="max-width: 400px; height: auto;"/>
```

Comprobemos el resultado en el navegador: deberíamos ver un dato curioso sobre perros.

Si hacemos un build y miramos los archivos generados en _./dist/index.html_, veremos que la imagen del perro ya está incluida, porque se obtuvo en el momento de la construcción del sitio.

```bash
npm run build
```

> Si estamos en modo **SSR**, este código se ejecutará en cada petición al servidor. Nunca se ejecuta en el navegador.

Y ahora te puede venir la siguiente duda... ¿Podemos ejecutar código en el navegador? ¡Por supuesto! Incluso podemos usar React, Vue o Svelte para ello.

Hagamos un ejemplo muy simple en vanilla JavaScript: añadiremos un botón que obtenga y muestre un dato curioso sobre gatos. El botón se llamará **“Get Cat Image”**.

_./src/pages/index.astro_

```diff
<html lang="en">
	<head>
		<meta charset="utf-8" />
		<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
		<meta name="viewport" content="width=device-width" />
		<meta name="generator" content={Astro.generator} />
		<title>Astro</title>
	</head>
	<body>
		<h1>{title}</h1>
		<img src={dogImageUrl} alt="Random Dog" />
+  <div>
+		  <button id="cat-image-button">Get Cat Image</button>
+  </div>
+  <div>
+		  <img id="cat-image" style="max-width: 400px; height: auto;"/>
+   </div>
	</body>
</html>

+ <script>
+ const button = document.getElementById("cat-image-button");
+ const imageEl = document.getElementById("cat-image") as HTMLImageElement;
+
+ if (button && imageEl) {
+  button.addEventListener("click", async () => {
+    const res = await fetch("https://api.thecatapi.com/v1/images/search");
+    const data = await res.json();
+    imageEl.src = data[0].url;
+  });
+}
+ </script>
```

Si lo ejecutamos, veremos que al hacer clic en el botón aparece un dato curioso sobre gatos.

```bash
npm run dev
```

Ahora quizá te preguntes: ¿cómo puedo depurar esto?

Veamos cómo depurar el código dentro de los **fences** y también el código del navegador.

Para depurar **código dentro de un fence**:

- Coloca un punto de ruptura (breakpoint) dentro del bloque de código.
- Abre una terminal en modo **JavaScript Debug Terminal** y ejecuta:

```bash
npm run dev
```

Cuando ejecutes el servidor, se detendrá en el punto de ruptura y podrás depurar.

Importante: en modo desarrollo local, cada vez que recargues la página el código del fence se ejecutará de nuevo. Pero esto solo ocurre en modo dev — en producción se ejecuta una sola vez, al construir el sitio.

¿Y cómo depuramos el **código del navegador**? Como siempre: con las DevTools del navegador.

**Bonus** Puedes extraer este código a un archivo _ts_, pero tendrás que ajustar un poco el código:

_./src/pages/cat.ts_

```ts
async function getCatImage() {
  const res = await fetch("https://api.thecatapi.com/v1/images/search");
  const data = await res.json();
  return data[0].url;
}

export const setupCatFactButton = () => {
  const button = document.getElementById("cat-image-button");
  const imageEl = document.getElementById("cat-image") as HTMLImageElement;

  if (button && imageEl) {
    button.addEventListener("click", async () => {
      const fact = await getCatImage();
      imageEl.src = fact;
    });
  }
};
```

_./src/pages/index.astro_

```diff
// (...)

<script>
+ import { setupCatFactButton } from "./cat";
+ setupCatFactButton();
- const button = document.getElementById("cat-image-button");
- const imageEl = document.getElementById("cat-image") as HTMLImageElement;
-
- if (button && imageEl) {
-  button.addEventListener("click", async () => {
-    const res = await fetch("https://api.thecatapi.com/v1/images/search");
-    const data = await res.json();
-    imageEl.src = data[0].url;
-  });
-}
</script>

```
