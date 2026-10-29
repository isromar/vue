# Versiones del proyecto

Este repositorio mantiene dos versiones de la aplicación en ramas separadas:

- `main`: versión original.
- `vue-version-formato-moderno`: versión con el diseño moderno. Su aplicación está en `to-do-list/`.

Cada rama conserva sus propios archivos y su propio README. Los cambios hechos en una rama no modifican la otra hasta que se fusionan; la versión moderna no necesita fusionarse con `main` para publicarse por separado.

## Ejecutar la versión moderna

Desde la carpeta `to-do-list/`:

```sh
npm install
npm run dev
```

Para generar los archivos de producción:

```sh
npm run build
```

La carpeta `to-do-list/dist/` resultante está configurada para publicarse bajo `/todolistia/`. Copia su contenido a `httpdocs/todolistia/` en el servidor.

Consulta [to-do-list/README.md](./to-do-list/README.md) para más detalles.
