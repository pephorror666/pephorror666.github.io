<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="theme-color" content="#0f172a">
    <title>Subestaciones e-distribución | Pro v2.1</title>

    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap');

        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background-color: #f1f5f9;
        }

        .glass-card {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.4);
        }
    </style>
</head>

<body class="p-4 md:p-8">

<div class="max-w-7xl mx-auto">

    <!-- HEADER -->
    <header class="mb-8 flex flex-col gap-6">
        <div>
            <div class="flex items-center gap-3 mb-2">
                <div class="bg-blue-600 p-2 rounded-xl shadow-lg shadow-blue-200">
                    <i data-lucide="zap" class="text-white w-6 h-6"></i>
                </div>
                <h1 class="text-3xl font-extrabold text-slate-900 tracking-tight">
                    Meteo <span class="text-blue-600 italic">Subestaciones</span>
                </h1>
            </div>
            <p class="text-slate-500 font-medium ml-1">
                Monitorización técnica y climática profesional.
            </p>
        </div>

        <div class="flex items-center gap-4 bg-white p-2 rounded-2xl shadow-sm border border-slate-200 w-fit">
            <div id="statusIndicator" class="flex items-center gap-2 px-3">
                <span class="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span>
                <span class="text-xs font-bold text-slate-500 uppercase tracking-widest">
                    Sincronizando...
                </span>
            </div>

            <label class="cursor-pointer bg-slate-900 hover:bg-black text-white px-5 py-2.5 rounded-xl text-sm font-semibold transition-all flex items-center gap-2 active:scale-95 shadow-md">
                <i data-lucide="upload" class="w-4 h-4"></i>
                Actualizar CSV
                <input type="file" id="csvInput" class="hidden" accept=".csv">
            </label>
        </div>
    </header>

    <!-- GRID -->
    <div class="grid grid-cols-1 xl:grid-cols-12 gap-6">

        <!-- LEFT PANEL (FILTERS ONLY) -->
        <div class="xl:col-span-4 space-y-6">

            <div class="glass-card p-5 md:p-8 rounded-3xl shadow-xl shadow-slate-200/50">
                <div class="space-y-6">

                    <div>
                        <label class="block text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] mb-3 ml-1">
                            Región
                        </label>
                        <select id="caSelect"
                                class="w-full bg-slate-50 ring-1 ring-slate-200 rounded-2xl px-4 py-3.5 text-slate-700 focus:ring-2 focus:ring-blue-500 outline-none font-medium">
                            <option value="">Esperando datos...</option>
                        </select>
                    </div>

                    <div>
                        <label class="block text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] mb-3 ml-1">
                            Buscador
                        </label>
                        <div class="relative">
                            <input type="text" id="subSearch"
                                   placeholder="Nombre de la instalación..."
                                   list="subList"
                                   class="w-full bg-slate-50 ring-1 ring-slate-200 rounded-2xl pl-12 pr-4 py-3.5 text-slate-700 focus:ring-2 focus:ring-blue-500 outline-none font-medium">
                            <i data-lucide="search"
                               class="w-5 h-5 absolute left-4 top-3.5 text-slate-400"></i>
                            <datalist id="subList"></datalist>
                        </div>
                    </div>

                </div>
            </div>

        </div>

        <!-- RIGHT PANEL -->
        <div class="xl:col-span-8 space-y-8">

            <!-- INFO CARD -->
            <div id="infoCard" class="hidden">
                <div class="glass-card rounded-3xl p-5 md:p-8 shadow-2xl shadow-blue-900/5">

                    <div class="flex flex-col md:flex-row justify-between gap-8">

                        <div class="flex-1">
                            <span class="inline-block bg-blue-100 text-blue-700 text-[10px] font-black uppercase tracking-widest px-3 py-1 rounded-lg mb-4">
                                Detalles Técnicos
                            </span>

                            <h2 id="displaySubName"
                                class="text-3xl md:text-4xl font-extrabold text-slate-900 mb-2">
                                ---
                            </h2>

                            <p id="displayLoc"
                               class="text-slate-500 font-medium mb-8">
                                ---
                            </p>

                            <div class="grid grid-cols-2 sm:grid-cols-3 gap-6">

                                <div>
                                    <span class="text-[10px] font-bold text-slate-400 uppercase">
                                        Tensión
                                    </span>
                                    <p id="displayVolt"
                                       class="text-lg font-bold text-blue-600">
                                        ---
                                    </p>
                                </div>

                                <div>
                                    <span class="text-[10px] font-bold text-slate-400 uppercase">
                                        Coord X
                                    </span>
                                    <p id="displayX"
                                       class="font-mono font-semibold text-slate-700">
                                        ---
                                    </p>
                                </div>

                                <div>
                                    <span class="text-[10px] font-bold text-slate-400 uppercase">
                                        Coord Y
                                    </span>
                                    <p id="displayY"
                                       class="font-mono font-semibold text-slate-700">
                                        ---
                                    </p>
                                </div>

                            </div>
                        </div>

                        <div class="flex flex-row md:flex-col gap-3">
                            <button onclick="openMap('gmaps')"
                                    class="flex-1 bg-slate-900 hover:bg-black text-white px-6 py-4 rounded-2xl font-bold">
                                Maps
                            </button>
                            <button onclick="openMap('osm')"
                                    class="flex-1 bg-white border-2 border-slate-100 text-slate-700 px-6 py-4 rounded-2xl font-bold">
                                OSM
                            </button>
                        </div>

                    </div>
                </div>
            </div>

            <!-- WEATHER CARD -->
            <div id="weatherCard" class="hidden">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">

                    <div class="bg-gradient-to-br from-blue-600 to-indigo-700 rounded-3xl p-6 md:p-8 text-white">
                        <p class="text-[10px] uppercase tracking-[0.2em] mb-6">
                            Tiempo Actual
                        </p>

                        <div class="flex items-center gap-4 mb-8">
                            <span id="weatherIcon" class="text-5xl md:text-6xl">☀️</span>
                            <h3 id="weatherTemp" class="text-4xl md:text-5xl font-black">
                                --
                            </h3>
                        </div>

                        <p id="weatherDesc" class="text-lg font-bold mb-2">
                            ---
                        </p>

                        <p id="weatherWind" class="text-sm">
                            ---
                        </p>

                        <p id="weatherTime" class="text-[10px] uppercase opacity-70 mt-4">
                            ---
                        </p>
                    </div>

                    <div class="md:col-span-2 glass-card rounded-3xl p-5 md:p-8">
                        <h4 class="text-slate-400 text-[10px] font-black uppercase tracking-[0.2em] mb-6">
                            Previsión Horaria
                        </h4>

                        <div id="forecastRow"
                             class="grid grid-cols-2 sm:grid-cols-4 gap-3">
                        </div>
                    </div>

                </div>
            </div>

            <!-- EMPTY STATE -->
            <div id="emptyState"
                 class="flex flex-col items-center justify-center py-24 text-slate-300">
                <p class="font-bold text-lg">
                    Selecciona una instalación
                </p>
                <p class="text-sm opacity-60">
                    Usa la región y el buscador superior
                </p>
            </div>

        </div>

    </div>
</div>

<script>
    lucide.createIcons();

    const DEFAULT_CSV_URL = "./subestaciones_edist.csv";

    const UTM_ZONE_BY_CA = {
        "01 - Andalucía": 30,
        "02 - Aragón": 30,
        "04 - Illes Balears": 31,
        "05 - Canarias": 28,
        "07 - C.León": 30,
        "08 - C.La Mancha": 30,
        "09 - Catalunya": 31,
    };

    const WMO_CODES = {
        0: ["Despejado", "☀️"],
        1: ["Casi despejado", "🌤️"],
        2: ["Parcialmente nuboso", "⛅"],
        3: ["Nublado", "☁️"],
        45: ["Niebla", "🌫️"],
        61: ["Lluvia", "🌧️"],
        71: ["Nieve", "❄️"],
        95: ["Tormenta", "⛈️"]
    };

    let rawData = [];
    let groupedData = {};
    let currentStation = null;

    // ---------- AUTO LOAD ----------
    window.addEventListener('DOMContentLoaded', fetchAutoCSV);

    async function fetchAutoCSV() {
        setLoadingStatus(true, "Cargando CSV local...");

        try {
            const response = await fetch(DEFAULT_CSV_URL + "?t=" + Date.now());

            if (!response.ok) throw new Error("No se pudo cargar el CSV");

            const text = await response.text();

            Papa.parse(text, {
                header: false,
                delimiter: ";",
                skipEmptyLines: true,
                complete: results => {
                    processData(results.data);
                    setLoadingStatus(false, "Datos cargados", "bg-green-500");
                },
                error: () => {
                    setLoadingStatus(false, "Error CSV", "bg-red-500");
                }
            });

        } catch (err) {
            console.warn("Auto load failed:", err);
            setLoadingStatus(false, "Carga manual requerida", "bg-amber-500");
        }
    }

    // ---------- MANUAL LOAD ----------
    document.getElementById('csvInput').addEventListener('change', function(e) {
        const file = e.target.files[0];
        if (!file) return;

        Papa.parse(file, {
            header: false,
            delimiter: ";",
            skipEmptyLines: true,
            complete: results => {
                processData(results.data);
                setLoadingStatus(false, "Archivo cargado", "bg-green-500");
            }
        });
    });

    // ---------- DATA PROCESSING ----------
    function processData(rows) {

        if (!rows || rows.length === 0) return;

        const firstRowIsHeader =
            isNaN(parseFloat(rows[0][0]?.toString().replace(',', '.')));

        const data = firstRowIsHeader ? rows.slice(1) : rows;

        rawData = data.map(row => ({
            x: parseFloat(row[0]?.toString().replace(',', '.')),
            y: parseFloat(row[1]?.toString().replace(',', '.')),
            volt: row[2],
            ca: row[3],
            sub: row[4],
            prov: row[5],
            mun: row[6]
        })).filter(r => r.sub && !isNaN(r.x));

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
        select.innerHTML = '<option value="">Elige región...</option>';

        cas.forEach(ca => {
            const opt = document.createElement('option');
            opt.value = ca;
            opt.innerText = ca;
            select.appendChild(opt);
        });

        updateRegionalList();
    }

    function updateRegionalList() {
        const ca = document.getElementById('caSelect').value;
        const dataList = document.getElementById('subList');
        dataList.innerHTML = '';

        Object.values(groupedData)
            .filter(s => s.ca === ca)
            .sort((a,b) => a.sub.localeCompare(b.sub))
            .forEach(s => {
                const opt = document.createElement('option');
                opt.value = s.sub;
                dataList.appendChild(opt);
            });
    }

    document.getElementById('caSelect')
        .addEventListener('change', updateRegionalList);

    document.getElementById('subSearch')
        .addEventListener('input', function(e) {
            const ca = document.getElementById('caSelect').value;
            const key = `${ca}|${e.target.value}`;
            if (groupedData[key]) showStation(groupedData[key]);
        });

    // ---------- SHOW STATION ----------
    async function showStation(station) {

        currentStation = station;

        document.getElementById('emptyState').classList.add('hidden');
        document.getElementById('infoCard').classList.remove('hidden');
        document.getElementById('weatherCard').classList.remove('hidden');

        document.getElementById('displaySubName').innerText = station.sub;
        document.getElementById('displayLoc').innerText =
            `${station.mun} (${station.prov})`;

        document.getElementById('displayVolt').innerText =
            Array.from(station.volts).sort((a,b)=>a-b).join(" / ") + " kV";

        document.getElementById('displayX').innerText =
            station.x.toLocaleString('es-ES', { minimumFractionDigits: 2 });

        document.getElementById('displayY').innerText =
            station.y.toLocaleString('es-ES', { minimumFractionDigits: 2 });

        const zone = UTM_ZONE_BY_CA[station.ca] || 30;
        const [lat, lon] = utmToLatLon(station.x, station.y, zone);

        currentStation.lat = lat;
        currentStation.lon = lon;

        fetchWeather(lat, lon);
    }

    // ---------- WEATHER ----------
    async function fetchWeather(lat, lon) {

    const url =
        `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,weathercode&forecast_hours=6`;

    try {
        const resp = await fetch(url);
        const data = await resp.json();

        const cw = data.current_weather;
        const [desc, icon] =
            WMO_CODES[cw.weathercode] || ["Desconocido", "🌡️"];

        document.getElementById('weatherTemp').innerText =
            Math.round(cw.temperature) + "°";

        document.getElementById('weatherDesc').innerText = desc;
        document.getElementById('weatherIcon').innerText = icon;
        document.getElementById('weatherWind').innerText =
            cw.windspeed + " km/h";

        document.getElementById('weatherTime').innerText =
            "Reporte: " + cw.time.split("T")[1] + "h";

        // HOURLY FORECAST
        const forecastRow = document.getElementById('forecastRow');
        forecastRow.innerHTML = '';

        for (let i = 1; i <= 4; i++) {
            const time = data.hourly.time[i].split("T")[1];
            const temp = Math.round(data.hourly.temperature_2m[i]);
            const [_, fIcon] =
                WMO_CODES[data.hourly.weathercode[i]] || ["-", "❓"];

            forecastRow.innerHTML += `
                <div class="flex flex-col items-center justify-center p-3 rounded-3xl bg-slate-50 border border-slate-100 shadow-sm">
                    <span class="text-[10px] text-slate-400 font-black mb-2">${time}h</span>
                    <span class="text-2xl mb-2">${fIcon}</span>
                    <span class="text-sm font-black text-slate-800">${temp}°</span>
                </div>
            `;
        }

    } catch (e) {
        console.error("Weather error:", e);
    }
}

    // ---------- MAP ----------
    function openMap(type) {
        if (!currentStation) return;

        const { lat, lon } = currentStation;

        const url = type === 'gmaps'
            ? `https://www.google.com/maps/search/?api=1&query=${lat},${lon}`
            : `https://www.openstreetmap.org/?mlat=${lat}&mlon=${lon}`;

        window.open(url, '_blank');
    }

    // ---------- STATUS ----------
    function setLoadingStatus(isLoading, text, color = "bg-amber-400") {
        document.getElementById('statusIndicator').innerHTML = `
            <span class="w-2 h-2 rounded-full ${color} ${isLoading ? 'animate-pulse' : ''}"></span>
            <span class="text-xs font-black uppercase">${text}</span>
        `;
    }

    // ---------- UTM ----------
    function utmToLatLon(easting, northing, zone) {
        const a = 6378137.0;
        const e = 0.081819191;
        const e1sq = 0.006739497;
        const k0 = 0.9996;

        const x = easting - 500000.0;
        const y = northing;

        const zone_cm = 6 * zone - 183.0;

        const m = y / k0;
        const mu = m / (a * (1 - e**2/4 - 3*e**4/64 - 5*e**6/256));

        const e1 = (1 - Math.sqrt(1 - e**2)) / (1 + Math.sqrt(1 - e**2));

        const j1 = 3*e1/2 - 27*e1**3/32;
        const j2 = 21*e1**2/16 - 55*e1**4/32;
        const j3 = 151*e1**3/96;
        const j4 = 1097*e1**4/512;

        const fp = mu +
            j1*Math.sin(2*mu) +
            j2*Math.sin(4*mu) +
            j3*Math.sin(6*mu) +
            j4*Math.sin(8*mu);

        const c1 = e1sq * Math.pow(Math.cos(fp), 2);
        const t1 = Math.pow(Math.tan(fp), 2);
        const n1 = a / Math.sqrt(1 - Math.pow(e*Math.sin(fp), 2));
        const r1 = a*(1-e**2)/Math.pow(1 - Math.pow(e*Math.sin(fp),2), 1.5);
        const d = x/(n1*k0);

        const lat = fp - (n1*Math.tan(fp)/r1) *
            (d**2/2 -
            (5 + 3*t1 + 10*c1 - 4*c1**2 - 9*e1sq)*d**4/24);

        const lon = (d -
            (1 + 2*t1 + c1)*d**3/6 +
            (5 - 2*c1 + 28*t1 - 3*c1**2 + 8*e1sq + 24*t1**2)*d**5/120)
            / Math.cos(fp);

        return [
            lat * 180/Math.PI,
            zone_cm + lon * 180/Math.PI
        ];
    }
</script>

</body>
</html>
