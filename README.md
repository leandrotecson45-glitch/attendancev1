<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance System</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body {
    margin: 0;
    font-family: Arial;
    background: #0f172a;
    color: white;
}

/* TOP BAR */
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

/* FULL MAP */
#map {
    height: calc(100vh - 60px);
    width: 100%;
}
</style>
</head>

<body>

<div class="topbar">
    <input type="text" id="name" placeholder="Enter Field Supervisor Name">
    <button class="timein" onclick="timeIn()">IN</button>
    <button class="timeout" onclick="timeOut()">OUT</button>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>

// ================= MAP INIT =================
const map = L.map('map').setView([15.5, 120.9], 14);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
}).addTo(map);

// ================= LOAD SAVED DATA =================
let records = JSON.parse(localStorage.getItem("attendance_data")) || [];

// show all saved pins
records.forEach(r => addMarker(r));

// ================= ADD MARKER =================
function addMarker(data) {
    const marker = L.marker([data.lat, data.lon]).addTo(map);

    marker.bindPopup(`
        <b>👤 ${data.name}</b><br>
        ${data.type}<br>
        🕒 ${data.time}<br>
        📍 ${data.lat.toFixed(5)}, ${data.lon.toFixed(5)}
    `);
}

// ================= GPS =================
function getLocation(callback) {
    if (!navigator.geolocation) {
        alert("GPS not supported");
        return;
    }

    navigator.geolocation.getCurrentPosition(
        (pos) => {
            callback(pos.coords.latitude, pos.coords.longitude);
        },
        (err) => {
            alert("Please enable GPS + location permission");
            console.log(err);
        },
        {
            enableHighAccuracy: true,
            timeout: 10000
        }
    );
}

// ================= SAVE =================
function save(type) {
    const name = document.getElementById("name").value;

    if (!name) {
        alert("Please enter name");
        return;
    }

    getLocation((lat, lon) => {

        const now = new Date();

        const record = {
            name: name,
            type: type,
            time: now.toLocaleString(),
            lat: lat,
            lon: lon
        };

        // save to storage
        records.push(record);
        localStorage.setItem("attendance_data", JSON.stringify(records));

        // add pin
        addMarker(record);

        // zoom to location
        map.setView([lat, lon], 17);
    });
}

// ================= BUTTONS =================
function timeIn() {
    save("TIME IN");
}

function timeOut() {
    save("TIME OUT");
}

</script>

</body>
</html>
