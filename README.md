
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance System</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css"/>

<style>
body {
    margin: 0;
    font-family: Arial;
    background: #0f172a;
    color: white;
}

.topbar {
    display: flex;
    gap: 5px;
    padding: 10px;
    background: #1e293b;
}

input {
    flex: 1;
    padding: 10px;
    border-radius: 8px;
    border: none;
}

button {
    padding: 10px;
    border: none;
    border-radius: 8px;
    color: white;
    font-weight: bold;
}

.timein { background: #22c55e; }
.timeout { background: #ef4444; }

#map {
    height: calc(100vh - 60px);
}
</style>
</head>

<body>

<div class="topbar">
    <input type="text" id="name" placeholder="Enter Name">
    <button class="timein" onclick="timeIn()">TIME IN</button>
    <button class="timeout" onclick="timeOut()">TIME OUT</button>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

<script>

// MAP INIT
const map = L.map('map').setView([15.5, 120.9], 14);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// 🔥 CLUSTER GROUP (KEY SOLUTION)
const markers = L.markerClusterGroup({
    spiderfyOnMaxZoom: true,
    showCoverageOnHover: false
});

map.addLayer(markers);

// STORAGE
let records = JSON.parse(localStorage.getItem("attendance_data")) || [];

// LOAD EXISTING
records.forEach(addMarker);

// ADD MARKER
function addMarker(data) {

    const marker = L.marker([data.lat, data.lon]);

    marker.bindPopup(`
        <b>👤 ${data.name}</b><br>
        ${data.type}<br>
        🕒 ${data.time}<br>
        📍 ${data.lat.toFixed(5)}, ${data.lon.toFixed(5)}
    `);

    markers.addLayer(marker); // 🔥 IMPORTANT
}

// GPS
function getLocation(callback) {
    navigator.geolocation.getCurrentPosition(
        pos => callback(pos.coords.latitude, pos.coords.longitude),
        err => alert("Enable GPS + allow permission"),
        { enableHighAccuracy: true }
    );
}

// SAVE
function save(type) {
    const name = document.getElementById("name").value;
    if (!name) return alert("Enter name");

    getLocation((lat, lon) => {

        const now = new Date();

        const record = {
            name,
            type,
            time: now.toLocaleString(),
            lat,
            lon
        };

        records.push(record);
        localStorage.setItem("attendance_data", JSON.stringify(records));

        addMarker(record);

        map.setView([lat, lon], 18);
    });
}

function timeIn() { save("TIME IN"); }
function timeOut() { save("TIME OUT"); }

</script>

</body>
</html>
