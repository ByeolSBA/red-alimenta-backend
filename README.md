# RED-ALIMENTA API

Backend REST para la fase 3 del proyecto académico RED-ALIMENTA. Implementa gestión de productos, donaciones y entregas con validación de stock y transacciones en base de datos MySQL.

## Requisitos

- Node.js 22 LTS
- npm 10+
- MySQL 8.0.x
- Prisma ORM 5.22.x

El proyecto se configuró para funcionar con un servidor MySQL local ejecutado en el equipo desde una instalación portable bajo `C:\Users\juanf\tools\mysql`.

## Instalación rápida

1. Abrir una terminal PowerShell en la carpeta del proyecto:
   ```powershell
   cd C:\Users\juanf\Red alimenta\backend
   ```
2. Asegurarse de que Node.js se pueda localizar en el PATH:
   ```powershell
   $env:Path = 'C:\Users\juanf\tools\node;' + $env:Path
   ```
3. Si PowerShell bloquea `npm.ps1`, usa explícitamente el ejecutable Windows:
   ```powershell
   & 'C:\Users\juanf\tools\node\npm.cmd' install
   ```
   En una instalación normal, también sirve:
   ```powershell
   npm install
   ```
4. Copiar el ejemplo de variables de entorno:
   ```powershell
   Copy-Item .env.example .env
   ```
5. Ajustar la cadena de conexión si hace falta:
   ```dotenv
   DATABASE_URL="mysql://redalimenta:change_me@127.0.0.1:3306/red_alimenta"
   PORT=4000
   ```

## Base de datos

### Iniciar MySQL local

```powershell
& 'C:\Users\juanf\tools\mysql\bin\mysqld.exe' --defaults-file='C:\Users\juanf\tools\mysql\my.ini' --console
```

### Crear base de datos y usuario

```powershell
& 'C:\Users\juanf\tools\mysql\bin\mysql.exe' -u root -e "CREATE DATABASE IF NOT EXISTS red_alimenta; CREATE USER IF NOT EXISTS 'redalimenta'@'localhost' IDENTIFIED BY 'change_me'; GRANT ALL PRIVILEGES ON red_alimenta.* TO 'redalimenta'@'localhost'; FLUSH PRIVILEGES;"
```

### Sincronizar esquema Prisma

```powershell
npx prisma generate
npx prisma db push
```

## Ejecutar la API

### Desarrollo

```powershell
npm run dev
```

### Producción

```powershell
npm run build
npm start
```

La API quedará disponible en:

- http://localhost:4000/health

## Endpoints

### Productos

- GET /api/products
- GET /api/products/:id
- POST /api/products
- PUT /api/products/:id
- DELETE /api/products/:id

Query params soportados:

- `search=arroz`
- `active=true`

### Donaciones

- GET /api/donations
- POST /api/donations

Query params soportados:

- `productId=1`

### Entregas

- GET /api/deliveries
- POST /api/deliveries

Query params soportados:

- `productId=1`

## Regla de negocio central

No se puede registrar una entrega si la cantidad supera el stock disponible.

- Donación válida: incrementa `stock_actual` y crea registro.
- Entrega válida: valida existencia, estado activo, cantidad positiva y stock suficiente, luego descuenta del inventario.
- Si la validación falla, responde `409 Conflict` con JSON con `success: false` y `error` legible.
- Las operaciones de inventario se ejecutan dentro de transacciones Prisma para evitar inconsistencias.

## Códigos HTTP

- 200 OK
- 201 Created
- 204 No Content
- 400 Bad Request
- 404 Not Found
- 409 Conflict
- 500 Internal Server Error

## Ejemplos de payload

### Crear producto

```json
{
  "nombre": "Arroz",
  "categoria": "Cereal",
  "unidad": "kg",
  "stock_actual": 10,
  "activo": true
}
```

### Registrar donación

```json
{
  "producto_id": 1,
  "cantidad": 5,
  "donante_referencia": "Banco de alimentos"
}
```

### Registrar entrega

```json
{
  "producto_id": 1,
  "cantidad": 7,
  "beneficiario_referencia": "Casa comunitaria"
}
```

## Colección de pruebas

La colección Postman importable está en:

- `tests/red-alimenta.postman_collection.json`

También se incluye un cliente REST de VS Code en:

- `tests/red-alimenta.http`

## Procedimiento de demostración

1. Arrancar MySQL.
2. Arrancar la API.
3. Crear producto.
4. Registrar donación.
5. Registrar entrega válida.
6. Intentar entrega con stock insuficiente.
7. Confirmar que la respuesta es `409` y que el stock no cambia.
8. Consultar el producto para verificar persistencia.

## Estructura del proyecto

```text
backend/
├── prisma/
│   ├── migrations/
│   └── schema.prisma
├── src/
│   ├── controllers/
│   ├── lib/
│   ├── middleware/
│   ├── routes/
│   ├── app.ts
│   └── server.ts
├── tests/
│   ├── red-alimenta.postman_collection.json
│   └── red-alimenta.http
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
├── README.md
└── package-lock.json
```
