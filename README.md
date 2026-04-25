### Hexlet tests and linter status:
[![Actions Status](https://github.com/Porico94/fullstack-javascript-project-138/actions/workflows/hexlet-check.yml/badge.svg)](https://github.com/Porico94/fullstack-javascript-project-138/actions)

[![Node.js CI](https://github.com/Porico94/fullstack-javascript-project-138/actions/workflows/ci.yml/badge.svg)](https://github.com/Porico94/fullstack-javascript-project-138/actions)

# 🌐 Page Loader
### Herramienta CLI para descargar páginas web completas con todos sus recursos locales

---

## 📖 Sobre el Proyecto

**Page Loader** es una utilidad de línea de comandos que descarga una página web y guarda todos sus recursos locales (imágenes, CSS, scripts, HTML canónico) en un directorio organizado. Las rutas dentro del HTML descargado se reescriben automáticamente para apuntar a los archivos locales, dejando la página funcional sin conexión a internet.

El proyecto simula el comportamiento de la opción *"Guardar página como..."* de un navegador, pero de forma programática y reutilizable como módulo Node.js. Fue construido con énfasis en **manejo de errores robusto**, **descargas concurrentes** y **testabilidad** mediante interceptores HTTP.

---

## 🛠️ Tecnologías Utilizadas

![JavaScript](https://img.shields.io/badge/JavaScript-ES2020-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js-CLI-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Jest](https://img.shields.io/badge/Jest-Testing-C21325?style=for-the-badge&logo=jest&logoColor=white)
![Axios](https://img.shields.io/badge/Axios-HTTP-5A29E4?style=for-the-badge&logo=axios&logoColor=white)

| Librería | Rol en el proyecto |
|---|---|
| `axios` | Peticiones HTTP para descargar la página y sus recursos |
| `cheerio` | Parseo y manipulación del HTML (equivalente a jQuery en Node.js) |
| `listr` | Muestra el progreso de descarga de cada recurso en la terminal |
| `commander` | Parseo de argumentos y opciones en la CLI |
| `debug` | Logs de depuración activables con variable de entorno |
| `nock` | Interceptor HTTP para simular respuestas de red en los tests |
| `jest` | Tests automatizados |

---

## ✨ Características Principales

- **Descarga completa** de una página web con todos sus recursos locales: imágenes, CSS, scripts y HTML canónico
- **Reescritura automática del HTML**: los atributos `src` y `href` de los recursos locales se reemplazan por rutas relativas al directorio de descarga
- **Descargas concurrentes** de todos los recursos usando `listr` con `{ concurrent: true }`
- **Nomenclatura determinista**: las URLs se convierten en nombres de archivo legibles (`codica-la-assets-application.css`)
- **Protección de rutas del sistema**: bloquea directorios como `/etc`, `/sys`, `/bin` para evitar escrituras peligrosas
- **Doble uso**: funciona como CLI (`page-loader <url>`) y como módulo importable (`import downloadPage from 'page-loader'`)
- **Manejo de errores explícito**: lanza errores descriptivos para HTTP 500, permisos denegados (`EACCES`) y directorios inexistentes (`ENOENT`)
- **Logs de depuración** activables con `DEBUG=page-loader`

---

## ⚙️ Instalación y Configuración

### Pre-requisitos

- `Node.js` v18 o superior
- `npm` v8 o superior

### Pasos

```bash
# 1. Clona el repositorio
git clone https://github.com/Porico94/Proyecto-Cargador-de-Paginas

# 2. Entra al directorio
cd Proyecto-Cargador-de-Paginas

# 3. Instala las dependencias
npm install

# 4. Vincula el ejecutable de forma global
npm link
```

### Uso como CLI

```bash
# Descarga en el directorio actual
page-loader https://ejemplo.com/pagina

# Descarga en un directorio específico
page-loader --output /ruta/destino https://ejemplo.com/pagina

# Ver ayuda
page-loader --help

# Activar logs de depuración
DEBUG=page-loader page-loader https://ejemplo.com/pagina
```

### Uso como módulo

```javascript
import downloadPage from 'page-loader';

const { filepath } = await downloadPage('https://ejemplo.com/pagina', './descargas');
console.log(`Página guardada en: ${filepath}`);
```

### Ejemplo de salida en terminal

```
  ✔ https://codica.la/assets/professions/nodejs.png
  ✔ https://codica.la/assets/application.css
  ✔ https://codica.la/packs/js/runtime.js
  ✔ https://codica.la/cursos.html

Page saved to: /home/usuario/descargas/codica-la-cursos.html
```

### Estructura de archivos generada

```
descargas/
├── codica-la-cursos.html          ← HTML con rutas reescritas
└── codica-la-cursos_files/        ← Directorio de recursos locales
    ├── codica-la-assets-application.css
    ├── codica-la-assets-professions-nodejs.png
    ├── codica-la-cursos.html
    └── codica-la-packs-js-runtime.js
```

### Ejecutar los tests

```bash
npm test
```

---

## 💡 Aprendizajes

> ### 🏆 El reto técnico clave: testear código que hace peticiones HTTP reales

El problema más difícil del proyecto no fue la lógica de descarga en sí, sino **cómo testearla de forma confiable sin depender de internet**. Al principio estaba ejecutando los tests haciendo peticiones HTTP reales a `codica.la`. El problema fue que los tests fallaban si no había conexión, si el servidor estaba caído, o si el contenido de la página cambiaba, factores completamente fuera de mi control.

La solución fue usar `nock`, una librería que intercepta las peticiones HTTP de `axios` antes de que salgan a la red y devuelve respuestas simuladas que yo mismo defino:

```javascript
// En el test, "engaña" a axios para que crea que habló con codica.la
nock('https://codica.la')
  .get('/cursos')
  .reply(200, originalHtmlContent)           // ← respuesta simulada del HTML
  .get('/assets/professions/nodejs.png')
  .reply(200, fakeImageData)                 // ← imagen falsa en buffer
  .get('/assets/application.css')
  .reply(200, fakeCssData);                  // ← CSS falso

// Ahora ejecuto la función real, y el test no sabe que la red está interceptada
await downloadPage('https://codica.la/cursos', tempDir);
```

Esto me enseñó el concepto de **mocking de dependencias externas**: aislar la unidad que quiero testear (mi lógica de descarga y reescritura del HTML) de sus dependencias no controlables (la red). El mismo principio que se usa en React con `jest.mock()` para simular llamadas a una API en tests de componentes.

Un segundo reto fue gestionar la **concurrencia de las descargas**. Al principio estaba descargando los recursos de forma secuencial con un loop. Luego me guiaron para reemplazarlo por `listr` con `{ concurrent: true }`, todas las descargas se ejecutan en paralelo usando Promises, lo que redujo significativamente el tiempo total en páginas con muchos recursos.

---

## 🧪 Cobertura de Tests

Los tests cubren tanto el flujo exitoso como los escenarios de error, usando `nock` para simular la red y directorios temporales reales del sistema operativo:

| Test | Escenario |
|---|---|
| ✅ Descarga completa | HTML + imágenes + CSS + scripts + HTML canónico |
| ❌ Error HTTP 500 | La petición principal falla con error de servidor |
| ❌ Permiso denegado al escribir | Intento de guardar en `/root` sin permisos |
| ❌ Error al crear directorio de recursos | `mkdir` falla por `EACCES` simulado con spy |
| ❌ Directorio de salida inexistente | Se pasa una ruta que no existe en el sistema |

---

## 📂 Estructura del Proyecto

```
page-loader/
├── bin/
│   └── page-loader.js           # Entry point CLI
├── src/
│   ├── index.js                 # Exporta downloadPage() para uso como módulo
│   ├── cli.js                   # Definición de comandos con commander
│   ├── page-loader.js           # Lógica principal: descarga, parseo y reescritura
│   └── utils.js                 # Helpers: urlToFilename, urlToDirname, sanitizeOutputDir
├── __tests__/
│   ├── page-loader.test.js      # Tests con Jest + nock
│   └── __fixtures__/
│       ├── test.html            # HTML original con rutas absolutas
│       └── expected.html        # HTML esperado con rutas reescritas
└── package.json
```

---

## 👤 Autor

**Pool Rimari** — Desarrollador Full-stack JavaScript

[![GitHub](https://img.shields.io/badge/GitHub-Porico94-181717?style=flat&logo=github)](https://github.com/Porico94)
