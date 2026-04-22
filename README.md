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

/* MAP FULL SCREEN */
#map {
    height: calc(100vh - 60px);
    width: 100%;
}
</style>
</head>

<body>

<div class="topbar">
    <input type="text" id="name" placeholder="Enter Field Supervisor Name">
    <button class="timein" onclick="timeIn()">TIME IN</button>
    <button class="timeout" onclick="timeOut()">TIME OUT</button>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>

// ================= MAP =================
const map = L.map('map').setView([15.5, 120.9], 14);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
}).addTo(map);

// ================= STORAGE =================
let records = JSON.parse(localStorage.getItem("attendance_data")) || [];

// LOAD EXISTING PINS
records.forEach(addMarker);

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

// ================= GPS FUNCTION =================
function getLocation(callback) {

    if (!navigator.geolocation) {
        alert("❌ GPS not supported sa device");
        return;
    }

    navigator.geolocation.getCurrentPosition(

        // SUCCESS
        (pos) => {
            callback(pos.coords.latitude, pos.coords.longitude);
        },

        // ERROR HANDLER
        (err) => {

            if (err.code === 1) {
                alert("❌ DENIED: I-enable mo Location permission sa browser");
            }
            else if (err.code === 2) {
                alert("❌ Hindi ma-detect location. I-ON mo GPS");
            }
            else if (err.code === 3) {
                alert("❌ Timeout. Mahina GPS signal");
            }
            else {
                alert("❌ Unknown GPS error");
            }

            console.log(err);
        },

        {
            enableHighAccuracy: true,
            timeout: 15000,
            maximumAge: 0
        }
    );
}

// ================= SAVE =================
function save(type) {
    const name = document.getElementById("name").value;

    if (!name) {
        alert("Enter name muna");
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

        // SAVE DATA
        records.push(record);
        localStorage.setItem("attendance_data", JSON.stringify(records));

        // ADD PIN
        addMarker(record);

        // FOCUS MAP
        map.setView([lat, lon], 17);

        alert("✅ " + type + " saved");
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
