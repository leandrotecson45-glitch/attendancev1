<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance Map</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body {
    margin: 0;
    font-family: Arial;
    background: #0f172a;
    color: white;
}

.topbar {
    padding: 10px;
    background: #1e293b;
    display: flex;
    gap: 5px;
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
    height: 90vh;
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

<script>
// INIT MAP
const map = L.map('map').setView([15.5, 120.9], 13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
}).addTo(map);

// LOAD DATA
let records = JSON.parse(localStorage.getItem("attendance_v2")) || [];

// LOAD ALL MARKERS
records.forEach(r => {
    addMarker(r);
});

// FUNCTION: ADD MARKER
function addMarker(data) {
    const marker = L.marker([data.lat, data.lon]).addTo(map);

    const label = `
        <b>${data.name}</b><br>
        ${data.type}<br>
        🕒 ${data.time}<br>
        📍 ${data.lat.toFixed(5)}, ${data.lon.toFixed(5)}
    `;

    marker.bindPopup(label);
}

// GET GPS
function getLocation(callback) {
    navigator.geolocation.getCurrentPosition(pos => {
        callback(pos.coords.latitude, pos.coords.longitude);
    });
}

// SAVE RECORD
function saveRecord(type, name, lat, lon) {
    const now = new Date();
    const time = now.toLocaleString();

    const record = {
        id: Date.now(),
        name,
        type,
        time,
        lat,
        lon
    };

    records.push(record);
    localStorage.setItem("attendance_v2", JSON.stringify(records));

    addMarker(record);

    map.setView([lat, lon], 16);

    alert(`${type} saved for ${name}`);
}

// TIME IN
function timeIn() {
    const name = document.getElementById("name").value;
    if (!name) return alert("Enter name");

    getLocation((lat, lon) => {
        saveRecord("TIME IN", name, lat, lon);
    });
}

// TIME OUT
function timeOut() {
    const name = document.getElementById("name").value;
    if (!name) return alert("Enter name");

    getLocation((lat, lon) => {
        saveRecord("TIME OUT", name, lat, lon);
    });
}
</script>

</body>
</html>
