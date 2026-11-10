<script setup>
import { ref, onMounted } from "vue";

// URL del backend en ECS Express. En Amplify se define como variable de entorno VITE_API_URL.
const API = (import.meta.env.VITE_API_URL || "http://localhost:8080").replace(/\/+$/, "");

const productos = ref([]);
const cargando = ref(false);
const guardando = ref(false);
const error = ref("");
const form = ref({ nombre: "", precio: "", stock: "" });

async function cargar() {
  cargando.value = true;
  error.value = "";
  try {
    const r = await fetch(`${API}/api/productos`);
    if (!r.ok) throw new Error(`Error ${r.status} al listar productos`);
    productos.value = await r.json();
  } catch (e) {
    error.value = e.message || "No se pudo conectar con la API";
  } finally {
    cargando.value = false;
  }
}

async function crear() {
  guardando.value = true;
  error.value = "";
  try {
    const r = await fetch(`${API}/api/productos`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        nombre: form.value.nombre,
        precio: Number(form.value.precio || 0),
        stock: Number(form.value.stock || 0),
      }),
    });
    const data = await r.json().catch(() => ({}));
    if (!r.ok) throw new Error(data.error || `Error ${r.status} al crear`);
    productos.value.unshift(data);
    form.value = { nombre: "", precio: "", stock: "" };
  } catch (e) {
    error.value = e.message;
  } finally {
    guardando.value = false;
  }
}

const soles = (n) => new Intl.NumberFormat("es-PE", { style: "currency", currency: "PEN" }).format(n);
const fecha = (s) => new Date(s).toLocaleString("es-PE");

onMounted(cargar);
</script>

<template>
  <main>
    <header>
      <h1>Productos</h1>
      <p class="api">API: <code>{{ API }}</code></p>
    </header>

    <form class="card" @submit.prevent="crear">
      <h2>Nuevo producto</h2>
      <div class="fields">
        <label>Nombre<input v-model="form.nombre" required maxlength="120" placeholder="Ej. Teclado" /></label>
        <label>Precio<input v-model="form.precio" type="number" min="0" step="0.01" required placeholder="0.00" /></label>
        <label>Stock<input v-model="form.stock" type="number" min="0" step="1" placeholder="0" /></label>
      </div>
      <button :disabled="guardando">{{ guardando ? "Guardando…" : "Crear producto" }}</button>
    </form>

    <p v-if="error" class="error">{{ error }}</p>

    <section class="card">
      <div class="row">
        <h2>Listado ({{ productos.length }})</h2>
        <button class="secondary" @click="cargar" :disabled="cargando">{{ cargando ? "Cargando…" : "Actualizar" }}</button>
      </div>
      <p v-if="!cargando && productos.length === 0" class="empty">Aún no hay productos.</p>
      <table v-else>
        <thead><tr><th>ID</th><th>Nombre</th><th class="num">Precio</th><th class="num">Stock</th><th>Creado</th></tr></thead>
        <tbody>
          <tr v-for="p in productos" :key="p.id">
            <td>{{ p.id }}</td><td>{{ p.nombre }}</td><td class="num">{{ soles(p.precio) }}</td>
            <td class="num">{{ p.stock }}</td><td>{{ fecha(p.creadoEn) }}</td>
          </tr>
        </tbody>
      </table>
    </section>
  </main>
</template>

<style>
* { box-sizing: border-box; }
body { margin: 0; font-family: system-ui, -apple-system, "Segoe UI", sans-serif; background: #f4f5f7; color: #1d2433; }
main { max-width: 880px; margin: 0 auto; padding: 24px 16px 48px; }
h1 { margin: 0 0 4px; }
h2 { margin: 0 0 12px; font-size: 1.1rem; }
.api { margin: 0 0 20px; color: #5b6475; font-size: .9rem; word-break: break-all; }
.card { background: #fff; border-radius: 10px; padding: 18px; margin-bottom: 16px; box-shadow: 0 1px 3px rgba(0,0,0,.08); }
.fields { display: grid; grid-template-columns: 2fr 1fr 1fr; gap: 12px; margin-bottom: 12px; }
label { display: flex; flex-direction: column; gap: 4px; font-size: .85rem; color: #5b6475; }
input { padding: 9px 10px; border: 1px solid #cfd4dc; border-radius: 6px; font-size: 1rem; }
button { background: #c8102e; color: #fff; border: 0; border-radius: 6px; padding: 9px 16px; font-size: .95rem; cursor: pointer; }
button.secondary { background: #e8eaee; color: #1d2433; }
button:disabled { opacity: .6; cursor: default; }
.row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.error { background: #fdecee; color: #a40e26; padding: 10px 14px; border-radius: 6px; }
.empty { color: #5b6475; }
table { width: 100%; border-collapse: collapse; font-size: .95rem; }
th, td { text-align: left; padding: 8px 6px; border-bottom: 1px solid #eceef2; }
.num { text-align: right; }
@media (max-width: 600px) { .fields { grid-template-columns: 1fr; } th:nth-child(5), td:nth-child(5) { display: none; } }
</style>
