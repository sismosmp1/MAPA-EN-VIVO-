# MAPA-EN-VIVO-
Mapa sísmico en vivio
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Mapa Sísmico</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <style>
    body, html { margin: 0; height: 100%; }
    #map { height: 100%; }
    .title {
      position: absolute; top: 10px; left: 10px;
      background: red; color: white;
      padding: 8px 12px; font-weight: bold; border-radius: 6px;
      z-index: 1000;
    }
    .timestamp {
      position: absolute; bottom: 10px; left: 10px;
      background: rgba(0, 0, 0, 0.7);
      color: white; padding: 6px 10px;
      font-size: 14px; border-radius: 6px;
      z-index: 1000;
    }
  </style>
</head>
<body>
  <div class="title">MAPA SÍSMICO</div>
  <div id="map"></div>
  <div id="timestamp" class="timestamp">Última actualización: ---</div>

  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script>
    const map = L.map('map').setView([23.6345, -102.5528], 5);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors'
    }).addTo(map);

    function actualizarHora() {
      const ahora = new Date();
      const hora = ahora.toLocaleTimeString("es-MX", { hour: '2-digit', minute: '2-digit', second: '2-digit' });
      document.getElementById('timestamp').textContent = "Última actualización: " + hora;
    }

    async function cargarSismos() {
      const res = await fetch('https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson');
      const data = await res.json();

      map.eachLayer(layer => {
        if (layer.sismosLayer) map.removeLayer(layer);
      });

      const group = L.layerGroup().addTo(map);
      group.sismosLayer = true;

      data.features.forEach(eq => {
        const [lon, lat] = eq.geometry.coordinates;
        const mag = eq.properties.mag;
        const place = eq.properties.place;

        L.circleMarker([lat, lon], {
          radius: mag * 2,
          color: 'red',
          fillColor: '#f03',
          fillOpacity: 0.6
        }).bindPopup(`<strong>${place}</strong><br>Magnitud: ${mag}`)
          .addTo(group);
      });

      actualizarHora();
    }

    cargarSismos();
    setInterval(cargarSismos, 60 * 1000); // cada 1 minuto
  </script>
</body>
</html>
