# Colecciones

¿Qué pasa si queremos mostrar una colección de elementos?

Pues es muy parecido a como lo hacemos con React: Podemos usar la función `map` de JavaScript para transformar un arreglo de datos en un arreglo de elementos de React.

Vamos a modificar la solicitud a la API para que devuelva varios datos.  
En este caso, vamos a solicitar 5 datos sobre perros.

```diff
---
- const res = await fetch("https://dogapi.dog/api/v2/facts");
+ const res = await fetch("https://dogapi.dog/api/v2/facts?limit=5");
const response = await res.json();
-const title = response?.data[0]?.attributes?.body ?? "Ooops api not working?";
+ const data = response?.data ?? [];
+ const facts : string[] = data.map((item: any) => item.attributes.body);
---
```

Y en el marcado:

```diff
  <body>
-    <h1>{title}</h1>
+   <h1>Hechos sobre perros</h1>
+   <ul>
+     {facts.map((fact : string) => (
+       <li>{fact}</li>
+     ))}
+   </ul>
    <button id="cat-fact-button">Obtener hecho de gato</button>
    <h2 id="cat-fact"></h2>
  </body>
```
