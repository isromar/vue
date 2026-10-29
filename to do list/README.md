# To-Do List App

Aplicación web de lista de tareas desarrollada con Vue 3 y Vite. Presenta un panel adaptable a móvil y escritorio para organizar tareas por prioridad, fecha y estado, y conserva la lista en el navegador.

## Vista previa

![Vista previa de la aplicación](https://raw.githubusercontent.com/isromar/vue/main/to%20do%20list/preview.JPG)

## Funcionalidades

- Añadir tareas con nombre, prioridad y fecha.
- Usar la fecha de hoy como valor predeterminado; se puede cambiar antes de añadir la tarea.
- Ordenar las tareas por prioridad: alta, media y baja.
- Cambiar la prioridad de una tarea ya creada.
- Marcar tareas como completadas o activas.
- Ver todas las tareas, solo las completadas o solo las activas.
- Filtrar tareas por nombre.
- Eliminar una tarea, todas las tareas completadas o la lista completa.
- Consultar los totales de tareas, completadas y activas.
- Ver el progreso de tareas completadas en el panel de resumen.
- Guardar y recuperar las tareas con `localStorage`.

## Tecnologías

- Vue 3
- Vite
- Bootstrap 5
- JavaScript, HTML y CSS

## Requisitos

- Node.js y npm instalados.

## Ejecutar en local

Desde la carpeta del proyecto (donde está `package.json`), instala las dependencias y arranca el servidor de desarrollo:

```sh
npm install
npm run dev
```

Vite mostrará en la terminal la dirección local para abrir la aplicación en el navegador. Para detener el servidor, pulsa `Ctrl+C` en esa terminal.

## Generar la versión de producción

```sh
npm run build
```

El comando genera los archivos listos para publicar dentro de `dist/`. Para probar esa versión localmente:

```sh
npm run preview
```

## Publicar en un hosting (por ejemplo, Plesk)

La aplicación está configurada para publicarse bajo la ruta `/todolist/`. Después de ejecutar `npm run build`, sube **el contenido de `dist/`** a la carpeta `todolist` de tu espacio web. Por ejemplo, si el dominio sirve sus archivos desde `httpdocs`, la estructura debería quedar así:

```text
httpdocs/
└── todolist/
    ├── index.html
    └── assets/
```

No es necesario subir `node_modules/` ni el código fuente para servir la versión compilada. Si cambias la configuración o el código, vuelve a generar la compilación y sube los archivos actualizados de `dist/`.

## Almacenamiento y limitaciones

Las tareas se guardan en `localStorage`, dentro del navegador. Por eso:

- se conservan al volver a abrir la aplicación en el mismo navegador;
- no se sincronizan entre dispositivos ni se comparten entre usuarios;
- borrar los datos del sitio en el navegador puede eliminar la lista.

La aplicación no necesita una base de datos ni un backend para sus funciones actuales. Para añadir cuentas, sincronización o almacenamiento compartido haría falta incorporar una API/backend y una base de datos o un servicio equivalente.
