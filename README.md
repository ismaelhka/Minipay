Minipay
// ====== index.js (Servidor Node.js + Express) ======
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middlewares
app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public')); // Carpeta con frontend

// Estado inicial
let saldo = 0;

// Endpoints
app.get('/saldo', (req, res) => {
  res.json({ saldo });
});

app.post('/depositar', (req, res) => {
  const { cantidad } = req.body;
  if (!cantidad || cantidad <= 0) return res.status(400).json({ error: 'Cantidad inválida' });
  saldo += cantidad;
  res.json({ saldo, mensaje: `Depositaste $${cantidad}` });
});

app.post('/retirar', (req, res) => {
  const { cantidad } = req.body;
  if (!cantidad || cantidad <= 0) return res.status(400).json({ error: 'Cantidad inválida' });
  if (cantidad > saldo) return res.status(400).json({ error: 'Saldo insuficiente' });
  saldo -= cantidad;
  // Aquí integrar PayPal Payouts o transferencia bancaria real usando variables de entorno
  res.json({ saldo, mensaje: `Retiraste $${cantidad} exitosamente` });
});

// Ruta catch-all para que Vercel no dé 404 en páginas internas
app.get('*', (req, res) => {
  res.sendFile(__dirname + '/public/index.html');
});

// Iniciar servidor
app.listen(PORT, () => console.log(`Servidor Minipay corriendo en http://localhost:${PORT}`));


// ====== public/index.html (Frontend simple) ======
/*
Crea carpeta 'public' en la raíz y coloca este archivo allí
*/

<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Minipay</title>
<style>
body { font-family: Arial; text-align: center; margin: 50px; }
input, button { padding: 10px; margin: 5px; }
#saldo { font-weight: bold; }
</style>
</head>
<body>
<h1>Minipay</h1>
<p>Saldo actual: $<span id="saldo">0</span></p>

<h3>Depositar</h3>
<input type="number" id="depositar" placeholder="Cantidad">
<button onclick="depositar()">Depositar</button>

<h3>Retirar</h3>
<input type="number" id="retirar" placeholder="Cantidad">
<button onclick="retirar()">Retirar</button>

<p id="mensaje"></p>

<script>
async function actualizarSaldo() {
  const res = await fetch('/saldo');
  const data = await res.json();
  document.getElementById('saldo').innerText = data.saldo;
}

async function depositar() {
  const cantidad = parseFloat(document.getElementById('depositar').value);
  const res = await fetch('/depositar', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ cantidad })
  });
  const data = await res.json();
  if(data.error) document.getElementById('mensaje').innerText = data.error;
  else { actualizarSaldo(); document.getElementById('mensaje').innerText = data.mensaje; }
}

async function retirar() {
  const cantidad = parseFloat(document.getElementById('retirar').value);
  const res = await fetch('/retirar', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ cantidad })
  });
  const data = await res.json();
  if(data.error) document.getElementById('mensaje').innerText = data.error;
  else { actualizarSaldo(); document.getElementById('mensaje').innerText = data.mensaje; }
}

actualizarSaldo();
</script>
</body>
</html>


// ====== .env (Variables de entorno, ejemplo) ======
/*
Crea archivo .env en la raíz con tus claves reales
PAYPAL_CLIENT_ID=tu_client_id
PAYPAL_CLIENT_SECRET=tu_client_secret
BANCO_EC_KEY=tu_clave_banco
PORT=3000
*/

// ====== package.json ======
/*
{
  "name": "minipay",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "body-parser": "^1.20.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1"
  }
}
*/
