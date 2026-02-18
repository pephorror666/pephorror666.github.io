<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Subestaciones e-distribución | Pro v2.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #f1f5f9; }
        .glass-card { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.4); }
        .custom-scroll::-webkit-scrollbar { width: 5px; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .loading-shimmer { background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%); background-size: 200% 100%; animation: shimmer 1.5s infinite; }
        @keyframes shimmer { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-7xl mx-auto">
        <header class="mb-10 flex flex-col md:flex-row md:items-end justify-between gap-6">
            <div>
                <div class="flex items-center gap-3 mb-2">
                    <div class="bg-blue-600 p-2 rounded-xl shadow-lg shadow-blue-200">
                        <i data-lucide="zap" class="text-white w-6 h-6"></i>
                    </div>
                    <h1 class="text-3xl font-extrabold text-slate-900 tracking-tight">Meteo <span class="text-blue-600 italic">Subestaciones</span></h1>
                </div>
                <p class="text-slate-500 font-medium ml-1">Monitorización técnica y climática profesional.</p>
            </div>
            
            <div class="flex items-center gap-4 bg-white p-2 rounded-2xl shadow-sm border border-slate-200">
                <div id="statusIndicator" class="flex items-center gap-2 px-3">
                    <span class="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span>
                    <span class="text-xs font-bold text-slate-500 uppercase tracking-widest">Sincronizando...</span>
                </div>
                <label class="cursor-pointer bg-slate-900 hover:bg-black text-white px-5 py-2.5 rounded-xl text-sm font-semibold transition-all flex items-center gap-2 active:scale-95 shadow-md">
                    <i data-lucide="upload" class="w-4 h-4"></i>
                    Actualizar CSV
                    <input type="file" id="csvInput" class="hidden" accept=".csv">
                </label>
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            <div class="lg:col-span-4 space-y-6">
                <div class="glass-card p-6 rounded-[2rem] shadow-xl shadow-slate-200/50">
                    <div class="space-y-5">
                        <div>
                            <label class="block text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] mb-3 ml-1">Región</label>
                            <select id="caSelect" class="w-full bg-slate-50 border-0 ring-1 ring-slate-200 rounded-2xl px-4 py-3.5 text-slate-700 focus:ring-2 focus:ring-blue-500 transition-all outline-none appearance-none cursor-pointer font-medium">
                                <option value="">Esperando datos...</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] mb-3 ml-1">Buscador Rápido</label>
                            <div class="relative">
                                <input type="text" id="subSearch" placeholder="Nombre de la instalación..." list="subList" class="w-full bg-slate-50 border-0 ring-1 ring-slate-200 rounded-2xl pl-12 pr-4 py-3.5 text-slate-700 focus:ring-2 focus:ring-blue-500 transition-all outline-none font-medium">
                                <i data-lucide="search" class="w-5 h-5 absolute left-4 top-3.5 text-slate-400"></i>
                                <datalist id="subList"></datalist>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="glass-card rounded-[2rem] shadow-xl shadow-slate-200/50 flex flex-col overflow-hidden max-h-[600px]">
                    <div class="p-6 border-b border-slate-100 flex justify-between items-center bg-white/50">
                        <h3 class="font-bold text-slate-800 flex items-center gap-2">
                            <i data-lucide="list" class="w-4 h-4 text-blue-500"></i> Listado Regional
                        </h3>
                        <span id="listCount" class="bg-blue-50 text-blue-600 text-xs font-black px-3 py-1 rounded-full border border-blue-100">0</span>
                    </div>
                    <div id="stationsList" class="overflow-y-auto custom-scroll p-3 divide-y divide-slate-50">
                        <div class="py-10 text-center space-y-3">
                            <div class="loading-shimmer h-4 w-3/4 mx-auto rounded"></div>
                            <div class="loading-shimmer h-4 w-1/2 mx-auto rounded"></div>
                            <p class="text-xs text-slate-400 font-medium">Cargando base de datos remota...</p>
                        </div>
                    </div>
                </div>
            </div>

            <div class="lg:col-span-8 space-y-8">
                <div id="infoCard" class="hidden transition-all duration-500 transform translate-y-4">
                    <div class="glass-card rounded-[2.5rem] p-8 shadow-2xl shadow-blue-900/5 relative overflow-hidden">
                        <div class="absolute -right-20 -top-20 w-64 h-64 bg-blue-50 rounded-full blur-3xl opacity-50"></div>
                        
                        <div class="relative flex flex-col md:flex-row justify-between gap-8">
                            <div class="flex-1">
                                <span id="displayBadge" class="inline-block bg-blue-100 text-blue-700 text-[10px] font-black uppercase tracking-widest px-3 py-1 rounded-lg mb-4">Detalles Técnicos</span>
                                <h2 id="displaySubName" class="text-4xl font-extrabold text-slate-900 mb-2 leading-tight">---</h2>
                                <p id="displayLoc" class="text-slate-500 font-medium flex items-center gap-2 mb-8">
                                    <i data-lucide="map-pin" class="w-4 h-4"></i> ---
                                </p>

                                <div class="grid grid-cols-2 sm:grid-cols-3 gap-6">
                                    <div class="space-y-1">
                                        <span class="text-[10px] font-bold text-slate-400 uppercase tracking-tighter">Niveles Tensión</span>
                                        <p id="displayVolt" class="text-lg font-bold text-blue-600">---</p>
                                    </div>
                                    <div class="space-y-1">
                                        <span class="text-[10px] font-bold text-slate-400 uppercase tracking-tighter">Coordenada X</span>
                                        <p id="displayX" class="text-md font-mono font-semibold text-slate-700">---</p>
                                    </div>
                                    <div class="space-y-1">
                                        <span class="text-[10px] font-bold text-slate-400 uppercase tracking-tighter">Coordenada Y</span>
                                        <p id="displayY" class="text-md font-mono font-semibold text-slate-700">---</p>
                                    </div>
                                </div>
                            </div>

                            <div class="flex flex-row md:flex-col gap-3 justify-end shrink-0">
                                <button onclick="openMap('gmaps')" class="flex-1 md:flex-none flex items-center justify-center gap-2 bg-slate-900 hover:bg-black text-white px-6 py-4 rounded-2xl transition-all shadow-lg hover:shadow-xl active:scale-95 font-bold">
                                    <img src="https://www.google.com/s2/favicons?domain=maps.google.com" class="w-4 h-4"> Maps
                                </button>
                                <button onclick="openMap('osm')" class="flex-1 md:flex-none flex items-center justify-center gap-2 bg-white border-2 border-slate-100 hover:bg-slate-50 text-slate-700 px-6 py-4 rounded-2xl transition-all active:scale-95 font-bold">
                                    <i data-lucide="map" class="w-4 h-4"></i> OSM
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <div id="weatherCard" class="hidden transition-all duration-700 transform translate-y-4">
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div class="md:col-span-1 bg-gradient-to-br from-blue-600 to-indigo-700 rounded-[2.5rem] p-8 text-white shadow-xl shadow-blue-200">
                            <p class="text-blue-100 text-[10px] font-black uppercase tracking-[0.2em] mb-6">Tiempo Actual</p>
                            <div class="flex items-center gap-4 mb-8">
                                <span id="weatherIcon" class="text-6xl drop-shadow-lg">☀️</span>
                                <h3 id="weatherTemp" class="text-5xl font-black leading-none tracking-tighter">--°</h3>
                            </div>
                            <p id="weatherDesc" class="text-xl font-bold mb-1 italic opacity-90">---</p>
                            <div class="flex flex-col gap-2 mt-10 pt-6 border-t border-white/10">
                                <p id="weatherWind" class="text-sm font-medium flex items-center gap-2">
                                    <i data-lucide="wind" class="w-4 h-4 opacity-70"></i> -- km/h
                                </p>
                                <p id="weatherTime" class="text-[10px] font-bold opacity-50 uppercase tracking-widest">---</p>
                            </div>
                        </div>

                        <div class="md:col-span-2 glass-card rounded-[2.5rem] p-8 shadow-xl shadow-slate-200/50">
                            <h4 class="text-slate-400 text-[10px] font-black uppercase tracking-[0.2em] mb-8 flex items-center gap-2">
                                <i data-lucide="clock" class="w-4 h-4"></i> Previsión Horaria
                            </h4>
                            <div id="forecastRow" class="grid grid-cols-4 gap-4 h-full items-center">
                                </div>
                        </div>
                    </div>
                </div>

                <div id="emptyState" class="flex flex-col items-center justify-center py-32 text-slate-300">
                    <div class="w-20 h-20 bg-slate-100 rounded-full flex items-center justify-center mb-6">
                        <i data-lucide="mouse-pointer-2" class="w-10 h-10 opacity-20"></i>
                    </div>
                    <p class="font-bold text-lg">Selecciona una instalación para monitorizar</p>
                    <p class="text-sm font-medium opacity-60">Explora el listado o usa el buscador superior</p>
                </div>
            </div>
        </div>
    </div>

    <script>
        // --- CONFIGURACIÓN ---
        const DRIVE_FILE_ID = "1OmiF_L21Dugwx55XhCIbWFjU6r91s8nA";
        // URL directa para descarga de archivos CSV en Drive
        //const DEFAULT_CSV_URL = `https://docs.google.com/spreadsheets/d/${DRIVE_FILE_ID}/export?format=csv`;
        //const DEFAULT_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQVhJfDZD6TJKW5Vf2AvM-mabsM42c7voy7FJjQCs8edFr4gUXdVhl9-hqJ2FYQ24LKRtKRdu37lToI/pub?output=csv';
        const DEFAULT_CSV_URL = 'https://docs.google.com/spreadsheets/d/1KM6QaAFo8AYVJQyi8b8IJ6XNAwv24aJ7qmrf8XSUbUs/edit?gid=1525832840#gid=1525832840';
        
        const UTM_ZONE_BY_CA = {
            "01 - Andalucía": 30, "02 - Aragón": 30, "04 - Illes Balears": 31,
            "05 - Canarias": 28, "07 - C.León": 30, "08 - C.La Mancha": 30, "09 - Catalunya": 30,
        };

        const WMO_CODES = {
            0: ["Despejado", "☀️"], 1: ["Casi despejado", "🌤️"], 2: ["Parcialmente nuboso", "⛅"],
            3: ["Nublado", "☁️"], 45: ["Niebla", "🌫️"], 51: ["Llovizna", "🌦️"], 
            61: ["Lluvia", "🌧️"], 71: ["Nieve", "❄️"], 95: ["Tormenta", "⛈️"]
        };

        let rawData = [];
        let groupedData = {};
        let currentStation = null;

        // --- INICIO ---
        lucide.createIcons();

        // Intentar carga automática al arrancar
        window.addEventListener('DOMContentLoaded', () => {
            fetchAutoCSV();
        });

        async function fetchAutoCSV() {
            setLoadingStatus(true, "Sincronizando con la nube...");
            try {
                // Añadimos un pequeño truco para evitar el caché del navegador
                const urlWithCacheBuster = DEFAULT_CSV_URL + (DEFAULT_CSV_URL.includes('?') ? '&' : '?') + 't=' + Date.now();
                
                const response = await fetch(urlWithCacheBuster);
                
                if (!response.ok) throw new Error("Respuesta de red no válida");
                
                const csvText = await response.text();
                
                // Verificamos que lo que recibimos sea realmente texto CSV y no una página de error
                if (csvText.includes("<!DOCTYPE html>") || csvText.length < 100) {
                    throw new Error("El enlace no devuelve un CSV válido. Revisa la publicación en la web.");
                }

                Papa.parse(csvText, {
                    header: false,
                    delimiter: ";",
                    skipEmptyLines: true,
                    complete: function(results) {
                        processData(results.data);
                        setLoadingStatus(false, "Datos sincronizados", "bg-green-500");
                    },
                    error: function(err) {
                        throw new Error("Error al procesar el formato CSV");
                    }
                });
            } catch (err) {
                console.error("Detalle del error:", err);
                setLoadingStatus(false, "Modo local", "bg-amber-500");
                
                // Mensaje amigable para el usuario en el listado
                document.getElementById('stationsList').innerHTML = `
                    <div class="py-10 text-center px-6">
                        <div class="bg-amber-50 border border-amber-100 p-4 rounded-2xl">
                            <p class="text-xs font-bold text-amber-700 mb-2 uppercase">⚠️ Conexión no disponible</p>
                            <p class="text-[11px] text-amber-600 leading-relaxed">
                                No hemos podido conectar con la base de datos de Google Drive debido a restricciones de seguridad (CORS) o enlace inválido.
                            </p>
                            <p class="text-[11px] text-amber-800 font-bold mt-3">Por favor, carga el archivo manualmente con el botón superior.</p>
                        </div>
                    </div>
                `;
            }
        }

        // --- MANEJO DE ARCHIVOS ---
        document.getElementById('csvInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;

            Papa.parse(file, {
                header: false,
                delimiter: ";",
                skipEmptyLines: true,
                complete: function(results) {
                    processData(results.data);
                    setLoadingStatus(false, "Archivo Local Cargado", "bg-green-500");
                }
            });
        });

        function processData(rows) {
            if (rows.length === 0) return;

            // Detectamos si la primera fila son cabeceras (si el primer elemento no empieza por número)
            const firstRowIsHeader = isNaN(parseFloat(rows[0][0].toString().replace(',', '.')));
            const dataToProcess = firstRowIsHeader ? rows.slice(1) : rows;

            // Mapeamos asumiendo el orden: X, Y, Tensión, CA, Subestación, Provincia, Municipio
            rawData = dataToProcess.map(row => ({
                x: parseFloat(row[0]?.toString().replace(',', '.')),
                y: parseFloat(row[1]?.toString().replace(',', '.')),
                volt: row[2],
                ca: row[3],
                sub: row[4],
                prov: row[5],
                mun: row[6]
            })).filter(r => r.sub && !isNaN(r.x));

            // Agrupar por CA y Subestación
            groupedData = {};
            rawData.forEach(r => {
                const key = `${r.ca}|${r.sub}`;
                if (!groupedData[key]) {
                    groupedData[key] = { ...r, volts: new Set() };
                }
                groupedData[key].volts.add(parseInt(r.volt));
            });

            const cas = [...new Set(rawData.map(r => r.ca))].sort();
            const select = document.getElementById('caSelect');
            select.innerHTML = '<option value="">Elige una región...</option>';
            cas.forEach(ca => {
                const opt = document.createElement('option');
                opt.value = ca; opt.innerText = ca;
                select.appendChild(opt);
            });

            updateRegionalList(); 
        }

        // --- UI HELPERS ---
        function setLoadingStatus(isLoading, text, colorClass = "bg-amber-400") {
            const indicator = document.getElementById('statusIndicator');
            indicator.innerHTML = `
                <span class="w-2 h-2 rounded-full ${colorClass} ${isLoading ? 'animate-pulse' : ''}"></span>
                <span class="text-xs font-black text-slate-500 uppercase tracking-widest">${text}</span>
            `;
        }

        function updateRegionalList() {
            const ca = document.getElementById('caSelect').value;
            const listContainer = document.getElementById('stationsList');
            const dataList = document.getElementById('subList');
            
            listContainer.innerHTML = '';
            dataList.innerHTML = '';

            const filtered = Object.values(groupedData)
                .filter(s => s.ca === ca)
                .sort((a,b) => a.sub.localeCompare(b.sub));

            document.getElementById('listCount').innerText = filtered.length;

            if (filtered.length === 0) {
                listContainer.innerHTML = '<p class="text-center text-slate-400 text-xs py-10 font-medium">Selecciona una región para ver instalaciones.</p>';
                return;
            }

            filtered.forEach(s => {
                const opt = document.createElement('option');
                opt.value = s.sub;
                dataList.appendChild(opt);

                const item = document.createElement('div');
                item.className = "p-4 hover:bg-white hover:shadow-md cursor-pointer transition-all rounded-2xl group flex justify-between items-center border border-transparent hover:border-slate-100 mb-1";
                item.innerHTML = `
                    <div class="flex items-center gap-3">
                        <div class="w-8 h-8 rounded-lg bg-slate-50 flex items-center justify-center group-hover:bg-blue-50 transition-colors">
                            <i data-lucide="zap" class="w-3.5 h-3.5 text-slate-400 group-hover:text-blue-500"></i>
                        </div>
                        <div>
                            <p class="font-bold text-slate-700 text-sm group-hover:text-blue-700">${s.sub}</p>
                            <p class="text-[9px] text-slate-400 font-bold uppercase tracking-tighter">${s.mun}</p>
                        </div>
                    </div>
                    <i data-lucide="arrow-right" class="w-4 h-4 text-slate-200 group-hover:text-blue-300 transform group-hover:translate-x-1 transition-all"></i>
                `;
                item.onclick = () => {
                    document.getElementById('subSearch').value = s.sub;
                    showStation(s);
                };
                listContainer.appendChild(item);
            });
            lucide.createIcons();
        }

        async function showStation(station) {
            currentStation = station;
            document.getElementById('emptyState').classList.add('hidden');
            const infoCard = document.getElementById('infoCard');
            const weatherCard = document.getElementById('weatherCard');
            
            infoCard.classList.remove('hidden');
            weatherCard.classList.remove('hidden');

            document.getElementById('displaySubName').innerText = station.sub;
            document.getElementById('displayLoc').innerText = `${station.mun} (${station.prov})`;
            document.getElementById('displayVolt').innerText = Array.from(station.volts).sort((a,b)=>a-b).join(' / ') + " kV";
            document.getElementById('displayX').innerText = station.x.toLocaleString('es-ES', {minimumFractionDigits: 2});
            document.getElementById('displayY').innerText = station.y.toLocaleString('es-ES', {minimumFractionDigits: 2});

            const zone = UTM_ZONE_BY_CA[station.ca] || 30;
            const [lat, lon] = utmToLatLon(station.x, station.y, zone);
            currentStation.lat = lat;
            currentStation.lon = lon;

            fetchWeather(lat, lon);
        }

        async function fetchWeather(lat, lon) {
            const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,weathercode&forecast_hours=6`;
            
            try {
                const resp = await fetch(url);
                const data = await resp.json();
                const cw = data.current_weather;
                const [desc, icon] = WMO_CODES[cw.weathercode] || ["Desconocido", "🌡️"];

                document.getElementById('weatherTemp').innerText = `${Math.round(cw.temperature)}°`;
                document.getElementById('weatherDesc').innerText = desc;
                document.getElementById('weatherIcon').innerText = icon;
                document.getElementById('weatherWind').innerText = `${cw.windspeed} km/h`;
                document.getElementById('weatherTime').innerText = `Reporte: ${cw.time.split('T')[1]}h`;

                const forecastRow = document.getElementById('forecastRow');
                forecastRow.innerHTML = '';
                for(let i=1; i<=4; i++) {
                    const time = data.hourly.time[i].split('T')[1];
                    const temp = Math.round(data.hourly.temperature_2m[i]);
                    const [_, fIcon] = WMO_CODES[data.hourly.weathercode[i]] || ["-", "❓"];
                    
                    forecastRow.innerHTML += `
                        <div class="flex flex-col items-center justify-center p-3 rounded-3xl bg-slate-50/50 border border-slate-100 shadow-sm">
                            <span class="text-[10px] text-slate-400 font-black mb-2">${time}h</span>
                            <span class="text-2xl mb-2">${fIcon}</span>
                            <span class="text-sm font-black text-slate-800">${temp}°</span>
                        </div>
                    `;
                }
            } catch (e) {
                console.error("Error clima:", e);
            }
        }

        // --- MAPAS Y FILTROS ---
        document.getElementById('caSelect').addEventListener('change', updateRegionalList);
        document.getElementById('subSearch').addEventListener('input', function(e) {
            const subName = e.target.value;
            const ca = document.getElementById('caSelect').value;
            const key = `${ca}|${subName}`;
            if (groupedData[key]) showStation(groupedData[key]);
        });

        function openMap(type) {
            if(!currentStation) return;
            const { lat, lon } = currentStation;
            const url = type === 'gmaps' 
                ? `https://www.google.com/maps/search/?api=1&query=${lat},${lon}`
                : `https://www.openstreetmap.org/?mlat=${lat}&mlon=${lon}#map=16/${lat}/${lon}`;
            window.open(url, '_blank');
        }

        function utmToLatLon(easting, northing, zone) {
            const a = 6378137.0;
            const e = 0.081819191;
            const e1sq = 0.006739497;
            const k0 = 0.9996;
            const x = easting - 500000.0;
            const y = northing;
            const zone_cm = 6 * zone - 183.0;
            const m = y / k0;
            const mu = m / (a * (1 - e**2 / 4.0 - 3 * e**4 / 64.0 - 5 * e**6 / 256.0));
            const e1 = (1.0 - Math.sqrt(1.0 - e**2)) / (1.0 + Math.sqrt(1.0 - e**2));
            const j1 = 3 * e1 / 2 - 27 * e1**3 / 32.0;
            const j2 = 21 * e1**2 / 16 - 55 * e1**4 / 32.0;
            const j3 = 151 * e1**3 / 96.0;
            const j4 = 1097 * e1**4 / 512.0;
            const phi1 = mu + j1 * Math.sin(2 * mu) + j2 * Math.sin(4 * mu) + j3 * Math.sin(6 * mu) + j4 * Math.sin(8 * mu);
            const c1 = e1sq * Math.pow(Math.cos(phi1), 2);
            const t1 = Math.pow(Math.tan(phi1), 2);
            const n1 = a / Math.sqrt(1 - Math.pow(e * Math.sin(phi1), 2));
            const r1 = a * (1 - e**2) / Math.pow(1 - Math.pow(e * Math.sin(phi1), 2), 1.5);
            const d = x / (n1 * k0);
            const lat = phi1 - (n1 * Math.tan(phi1) / r1) * (Math.pow(d, 2) / 2 - (5 + 3 * t1 + 10 * c1 - 4 * Math.pow(c1, 2) - 9 * e1sq) * Math.pow(d, 4) / 24);
            const lon = (d - (1 + 2 * t1 + c1) * Math.pow(d, 3) / 6 + (5 - 2 * c1 + 28 * t1 - 3 * Math.pow(c1, 2) + 8 * e1sq + 24 * Math.pow(t1, 2)) * Math.pow(d, 5) / 120) / Math.cos(phi1);
            return [lat * 180 / Math.PI, zone_cm + lon * 180 / Math.PI];
        }
    </script>
</body>
</html>
