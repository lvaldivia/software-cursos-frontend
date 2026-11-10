# Productos Web

Frontend Vue 3 + Vite. Instalar con `npm ci` y ejecutar con `npm run dev`.
Compilar con `npm run build`; Amplify usa `amplify.yml`.

Copiar `.env.example` a `.env.development.local` para desarrollo o configurar
VITE_API_URL en Amplify, con un valor por rama/entorno (dev, staging, prod).
Cambiar la URL requiere compilar y desplegar de nuevo. Las variables VITE_
son publicas en el navegador: nunca poner contrasenas ni tokens privados.

GitHub Actions valida el build y guarda `productos-web-<commit>` como artefacto.
Configurar la variable de repositorio VITE_API_URL para ese build de CI;
el build de Amplify usa su propia configuracion de variables.
Marcar versiones con tags Git, por ejemplo `v1.0.0`.
