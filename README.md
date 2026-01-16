<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SimuMed - Simulador Médico</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; background-color: #f0f8ff; color: #333; }
        .watermark { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%) rotate(-45deg); font-size: 120px; color: rgba(255, 0, 0, 0.1); z-index: 1; pointer-events: none; font-weight: bold; }
        .container { max-width: 1200px; margin: 0 auto; padding: 20px; position: relative; z-index: 2; }
        .header { background: linear-gradient(135deg, #4a90e2, #357abd); color: white; padding: 20px; border-radius: 10px; text-align: center; margin-bottom: 30px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .warning { background: #ff6b6b; color: white; padding: 15px; border-radius: 5px; margin-bottom: 20px; text-align: center; font-weight: bold; }
        .search-section, .result-section, .notes-section { background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 30px; }
        .search-box { display: flex; gap: 10px; margin-bottom: 20px; }
        input[type="text"], input[type="date"], textarea { flex: 1; padding: 12px; border: 2px solid #ddd; border-radius: 5px; font-size: 16px; }
        textarea { min-height: 100px; resize: vertical; }
        button { background: #4a90e2; color: white; border: none; padding: 12px 25px; border-radius: 5px; cursor: pointer; font-size: 16px; transition: background 0.3s; }
        button:hover { background: #357abd; }
        .patient-info { border-left: 4px solid #4a90e2; padding-left: 20px; margin-bottom: 30px; }
        .medical-record { background: #f8f9fa; padding: 20px; border-radius: 8px; margin: 15px 0; border: 1px solid #e9ecef; }
        .record-header { color: #4a90e2; font-weight: bold; font-size: 18px; margin-bottom: 10px; }
        .vitals { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin: 20px 0; }
        .vital-card { background: white; padding: 15px; border-radius: 8px; border-left: 4px solid #28a745; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .medications { background: #fff3cd; padding: 15px; border-radius: 8px; border-left: 4px solid #ffc107; }
        .allergies { background: #f8d7da; padding: 15px; border-radius: 8px; border-left: 4px solid #dc3545; }
        .note { border: 1px solid #e3e3e3; padding: 10px; margin-bottom: 10px; border-radius: 6px; background:#fff; }
        .note-header { display:flex; justify-content:space-between; align-items:center; gap:10px; margin-bottom:6px; }
        .note-title { font-weight:bold; color:#333; }
        .small { font-size:0.9rem; color:#666 }
        .actions { margin-top:10px; display:flex; gap:8px; flex-wrap:wrap }
        .btn-secondary { background:#6c757d }
    </style>
</head>
<body>
    <div class="watermark">SIMULACIÓN</div>
    <div class="container">
        <div class="header">
            <h1>🏥 SimuMed - Sistema de Simulación Médica</h1>
            <p>Herramienta educativa para practicar consultas médicas</p>
        </div>

        <div class="warning">
            ⚠️ ESTO ES UNA SIMULACIÓN - TODOS LOS DATOS SON FICTICIOS Y PARA USO EDUCATIVO ÚNICAMENTE
        </div>

        <div class="search-section">
            <h2>🔍 Búsqueda de Paciente</h2>
            <p>Introduce un número de tarjeta sanitaria ficticio (cualquier número de 10 dígitos)</p>
            <div class="search-box">
                <input type="text" id="tarjeta" placeholder="Ej: 1234567890" maxlength="10" aria-label="Tarjeta sanitaria">
                <button id="buscarBtn">Buscar Paciente</button>
                <button id="limpiarBtn" class="btn-secondary">Limpiar</button>
            </div>
            <div>
                <small>💡 Ejemplos: 1234567890, 9876543210, 5555555555</small>
            </div>
        </div>

        <div class="result-section" id="resultados" style="display:none;">
            <div class="patient-info" id="infoPersonal"></div>

            <div class="medical-record">
                <div class="record-header">📋 Historial Médico Reciente</div>
                <div id="historialMedico"></div>
            </div>

            <div class="vitals" id="signos"></div>

            <div class="medications">
                <h3>💊 Medicación Actual</h3>
                <div id="medicamentos"></div>
            </div>

            <div class="allergies">
                <h3>⚠️ Alergias Conocidas</h3>
                <div id="alergias"></div>
            </div>

            <div class="actions" style="margin-top:15px;">
                <button id="imprimirBtn">🖨️ Imprimir ficha</button>
            </div>
        </div>

        <div class="notes-section" id="notasSection" style="display:none;">
            <h2>📝 Notas / Información añadida</h2>

            <div style="display:flex; gap:10px; margin-bottom:12px;">
                <input id="notaTitulo" placeholder="Título de la nota" type="text" aria-label="Título de la nota">
                <input id="notaFecha" type="date" aria-label="Fecha de la nota">
            </div>

            <textarea id="notaTexto" placeholder="Escribe tu nota aquí..."></textarea>

            <div class="actions" style="margin-top:10px;">
                <button id="añadirNotaBtn">➕ Añadir nota</button>
                <button id="imprimirNotasBtn" class="btn-secondary">🖨️ Imprimir notas</button>
                <button id="borrarTodasBtn" class="btn-secondary">🗑️ Borrar todas las notas (local)</button>
            </div>

            <div id="listaNotas" style="margin-top:20px;"></div>
        </div>

    </div>

    <script>
        // Datos iniciales (como antes)
        const nombres = ["Ana García", "Carlos Rodríguez", "María López", "José Martínez", "Carmen Sánchez", "Antonio Pérez", "Isabel Ruiz", "Francisco Moreno", "Laura Jiménez", "Manuel Álvarez"];
        const apellidos = ["González", "Fernández", "López", "Martínez", "Sánchez", "Pérez", "Gómez", "Martín", "Jiménez", "Ruiz"];
        const condiciones = ["Hipertensión leve", "Diabetes tipo 2 controlada", "Asma bronquial", "Artritis reumatoide", "Migraña crónica", "Ansiedad", "Reflujo gastroesofágico"];
        const medicamentos = ["Ibuprofeno 400mg", "Paracetamol 500mg", "Omeprazol 20mg", "Enalapril 10mg", "Metformina 850mg", "Salbutamol inhalador"];
        const alergias = ["Polen", "Penicilina", "Frutos secos", "Ácaros", "Ninguna conocida", "Látex"];

        // Semilla RNG simple (como antes)
        Math.seedrandom = function(seed) {
            return function() {
                const x = Math.sin(seed++) * 10000;
                return x - Math.floor(x);
            };
        };

        function generarDatos(tarjeta) {
            const seed = parseInt(tarjeta) || 1234567890;
            const random = Math.seedrandom(seed);
            return {
                nombre: nombres[Math.floor(random() * nombres.length)],
                apellido: apellidos[Math.floor(random() * apellidos.length)],
                edad: 25 + Math.floor(random() * 50),
                genero: random() > 0.5 ? "Masculino" : "Femenino",
                presion: `${120 + Math.floor(random() * 40)}/${80 + Math.floor(random() * 20)}`,
                pulso: 60 + Math.floor(random() * 40),
                temperatura: (36 + random() * 2).toFixed(1),
                peso: 60 + Math.floor(random() * 40),
                altura: 150 + Math.floor(random() * 30),
                condicion: condiciones[Math.floor(random() * condiciones.length)],
                medicamento: medicamentos[Math.floor(random() * medicamentos.length)],
                alergia: alergias[Math.floor(random() * alergias.length)]
            };
        }

        // LocalStorage keys: "salud:<tarjeta>:notas"
        function notasKey(tarjeta) { return `salud:${tarjeta}:notas`; }

        function guardarNotas(tarjeta, notas) {
            localStorage.setItem(notasKey(tarjeta), JSON.stringify(notas));
        }
        function cargarNotas(tarjeta) {
            const raw = localStorage.getItem(notasKey(tarjeta));
            return raw ? JSON.parse(raw) : [];
        }

        // UI handlers
        let tarjetaActual = null;
        const buscarBtn = document.getElementById('buscarBtn');
        const limpiarBtn = document.getElementById('limpiarBtn');
        const imprimirBtn = document.getElementById('imprimirBtn');
        const añadirNotaBtn = document.getElementById('añadirNotaBtn');
        const imprimirNotasBtn = document.getElementById('imprimirNotasBtn');
        const borrarTodasBtn = document.getElementById('borrarTodasBtn');

        buscarBtn.addEventListener('click', buscarPaciente);
        limpiarBtn.addEventListener('click', limpiarBusqueda);
        imprimirBtn.addEventListener('click', () => imprimirFicha(tarjetaActual));
        añadirNotaBtn.addEventListener('click', añadirNota);
        imprimirNotasBtn.addEventListener('click', () => imprimirNotas(tarjetaActual));
        borrarTodasBtn.addEventListener('click', borrarTodasNotas);

        document.getElementById('tarjeta').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') buscarPaciente();
        });

        function buscarPaciente() {
            const tarjeta = document.getElementById('tarjeta').value.trim();
            if (tarjeta.length < 10) return alert('Por favor, introduce un número de tarjeta de 10 dígitos');
            tarjetaActual = tarjeta;
            const datos = generarDatos(tarjeta);
            mostrarResultados(datos, tarjeta);
            cargarYMostrarNotas(tarjeta);
            document.getElementById('notasSection').style.display = 'block';
        }

        function limpiarBusqueda() {
            tarjetaActual = null;
            document.getElementById('tarjeta').value = '';
            document.getElementById('resultados').style.display = 'none';
            document.getElementById('notasSection').style.display = 'none';
            document.getElementById('listaNotas').innerHTML = '';
        }

        function mostrarResultados(datos, tarjeta) {
            document.getElementById('infoPersonal').innerHTML = `
                <h2>👤 Información Personal</h2>
                <p><strong>Nombre:</strong> ${datos.nombre} ${datos.apellido}</p>
                <p><strong>Tarjeta Sanitaria:</strong> ${tarjeta}</p>
                <p><strong>Edad:</strong> ${datos.edad} años</p>
                <p><strong>Género:</strong> ${datos.genero}</p>
                <p><strong>Fecha de consulta:</strong> ${new Date().toLocaleDateString('es-ES')}</p>
            `;

            document.getElementById('historialMedico').innerHTML = `
                <p><strong>Condición actual:</strong> ${datos.condicion}</p>
                <p><strong>Última consulta:</strong> ${new Date(Date.now() - Math.random() * 30 * 24 * 60 * 60 * 1000).toLocaleDateString('es-ES')}</p>
                <p><strong>Notas:</strong> Paciente en seguimiento regular. Evolución favorable.</p>
            `;

            document.getElementById('signos').innerHTML = `
                <div class="vital-card">
                    <h4>🩺 Presión Arterial</h4><p>${datos.presion} mmHg</p>
                </div>
                <div class="vital-card">
                    <h4>💗 Pulso</h4><p>${datos.pulso} bpm</p>
                </div>
                <div class="vital-card">
                    <h4>🌡️ Temperatura</h4><p>${datos.temperatura}°C</p>
                </div>
                <div class="vital-card">
                    <h4>⚖️ Peso</h4><p>${datos.peso} kg</p>
                </div>
                <div class="vital-card">
                    <h4>📏 Altura</h4><p>${datos.altura} cm</p>
                </div>
            `;

            document.getElementById('medicamentos').innerHTML = `
                <p>• ${datos.medicamento} - 1 cada 8 horas</p>
                <p>• Vitamina D3 - 1 diaria</p>
                <p><small>Última actualización: ${new Date().toLocaleDateString('es-ES')}</small></p>
            `;

            document.getElementById('alergias').innerHTML = `
                <p><strong>Alergias:</strong> ${datos.alergia}</p>
                <p><small>Verificado en última consulta</small></p>
            `;

            document.getElementById('resultados').style.display = 'block';
            document.getElementById('resultados').scrollIntoView({ behavior: 'smooth' });
        }

        // Notas: añadir, listar, borrar
        function cargarYMostrarNotas(tarjeta) {
            const notas = cargarNotas(tarjeta);
            renderNotas(notas);
        }

        function renderNotas(notas) {
            const lista = document.getElementById('listaNotas');
            lista.innerHTML = '';
            if (!notas || notas.length === 0) {
                lista.innerHTML = '<p class="small">No hay notas guardadas para este paciente.</p>';
                return;
            }
            notas.forEach((n, idx) => {
                const div = document.createElement('div');
                div.className = 'note';
                div.innerHTML = `
                    <div class="note-header">
                        <div>
                            <div class="note-title">${escapeHtml(n.titulo || 'Sin título')}</div>
                            <div class="small">${n.fecha || ''} · <span class="small">${escapeHtml(n.autor || 'Usuario local')}</span></div>
                        </div>
                        <div>
                            <button data-idx="${idx}" class="btn-eliminar">Eliminar</button>
                        </div>
                    </div>
                    <div>${escapeHtml(n.texto).replace(/\n/g,'<br>')}</div>
                `;
                lista.appendChild(div);
            });
            // attach delete handlers
            document.querySelectorAll('.btn-eliminar').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    const idx = parseInt(e.target.getAttribute('data-idx'));
                    eliminarNota(idx);
                });
            });
        }

        function añadirNota() {
            if (!tarjetaActual) return alert('Busca primero un paciente para asociar la nota.');
            const titulo = document.getElementById('notaTitulo').value.trim();
            const fecha = document.getElementById('notaFecha').value || new Date().toISOString().slice(0,10);
            const texto = document.getElementById('notaTexto').value.trim();
            if (!texto) return alert('La nota está vacía.');
            const notas = cargarNotas(tarjetaActual);
            notas.unshift({ titulo, fecha, texto, autor: 'Usuario local' });
            guardarNotas(tarjetaActual, notas);
            // limpiar campos
            document.getElementById('notaTitulo').value = '';
            document.getElementById('notaFecha').value = '';
            document.getElementById('notaTexto').value = '';
            renderNotas(notas);
        }

        function eliminarNota(idx) {
            if (!confirm('¿Eliminar esta nota?')) return;
            const notas = cargarNotas(tarjetaActual);
            notas.splice(idx, 1);
            guardarNotas(tarjetaActual, notas);
            renderNotas(notas);
        }

        function borrarTodasNotas() {
            if (!tarjetaActual) return alert('Busca primero un paciente.');
            if (!confirm('¿Borrar todas las notas locales de este paciente?')) return;
            localStorage.removeItem(notasKey(tarjetaActual));
            renderNotas([]);
        }

        // Imprimir ficha completa
        function imprimirFicha(tarjeta) {
            if (!tarjeta) return alert('Busca primero un paciente.');
            const patientHtml = document.getElementById('infoPersonal').innerHTML;
            const historial = document.getElementById('historialMedico').innerHTML;
            const signos = document.getElementById('signos').innerHTML;
            const meds = document.getElementById('medicamentos').innerHTML;
            const alrg = document.getElementById('alergias').innerHTML;
            const notas = cargarNotas(tarjeta);

            const printable = `
                <html>
                <head>
                    <meta charset="utf-8">
                    <title>Ficha paciente ${tarjeta}</title>
                    <style>
                        body{font-family:Arial,Helvetica,sans-serif;color:#222;padding:20px}
                        h1{color:#0b5ed7}
                        .section{margin-bottom:18px;padding-bottom:8px;border-bottom:1px solid #ddd}
                        .note{border:1px solid #ddd;padding:8px;border-radius:6px;margin-bottom:8px}
                        .small{color:#666;font-size:0.9rem}
                    </style>
                </head>
                <body>
                    <h1>Ficha paciente</h1>
                    <div class="section">
                        ${patientHtml}
                    </div>
                    <div class="section">
                        <h3>Historial</h3>${historial}
                    </div>
                    <div class="section">
                        <h3>Signos vitales</h3>${signos}
                    </div>
                    <div class="section">
                        <h3>Medicación</h3>${meds}
                    </div>
                    <div class="section">
                        <h3>Alergias</h3>${alrg}
                    </div>
                    <div class="section">
                        <h3>Notas añadidas</h3>
                        ${notas.length===0 ? '<p class="small">No hay notas.</p>' : notas.map(n => `<div class="note"><div><strong>${escapeHtml(n.titulo||'Sin título')}</strong> <span class="small">(${n.fecha})</span></div><div>${escapeHtml(n.texto).replace(/\\n/g,'<br>')}</div></div>`).join('')}
                    </div>
                    <script>window.print();</script>
                </body>
                </html>
            `;
            const w = window.open('', '_blank');
            w.document.open();
            w.document.write(printable);
            w.document.close();
        }

        function imprimirNotas(tarjeta) {
            if (!tarjeta) return alert('Busca primero un paciente.');
            const notas = cargarNotas(tarjeta);
            const printable = `
                <html><head><meta charset="utf-8"><title>Notas ${tarjeta}</title>
                <style>body{font-family:Arial;padding:20px}.note{border:1px solid #ddd;padding:8px;margin-bottom:8px;border-radius:6px}</style>
                </head><body>
                <h1>Notas del paciente ${tarjeta}</h1>
                ${notas.length===0 ? '<p>No hay notas.</p>' : notas.map(n=>`<div class="note"><div><strong>${escapeHtml(n.titulo||'Sin título')}</strong> <small>${n.fecha}</small></div><div>${escapeHtml(n.texto).replace(/\\n/g,'<br>')}</div></div>`).join('')}
                <script>window.print();</script>
                </body></html>`;
            const w = window.open('', '_blank');
            w.document.open();
            w.document.write(printable);
            w.document.close();
        }

        // pequeño escape para evitar inyección al mostrar HTML
        function escapeHtml(text) {
            if (!text && text !== 0) return '';
            return String(text)
                .replaceAll('&','&amp;')
                .replaceAll('<','&lt;')
                .replaceAll('>','&gt;')
                .replaceAll('"','&quot;')
                .replaceAll("'",'&#039;');
        }

        // Si el usuario comparte el enlace raw externo o abre la página con "tarjeta" en la query,
        // podríamos prellenar y buscar automáticamente. (Opcional)
        (function checkQuery() {
            const params = new URLSearchParams(location.search);
            const t = params.get('tarjeta');
            if (t) {
                document.getElementById('tarjeta').value = t;
                buscarPaciente();
            }
        })();
    </script>
</body>
</html>name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Upload artifact for GitHub Pages
        uses: actions/upload-pages-artifact@v1
        with:
          path: './'

      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v1

