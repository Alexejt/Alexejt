<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Preis- und Laufzeitberechnungstool</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
            max-width: 800px;
            margin: auto;
            background-color: #f1f1f1;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        input, select {
            width: 100%;
            padding: 10px;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 16px;
        }
        button {
            padding: 15px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 18px;
            width: 100%;
        }
        button:hover {
            background-color: #0056b3;
        }
        .result {
            margin-top: 20px;
            background-color: #f8f9fa;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Preis- und Laufzeitberechnungstool</h1>
        <div class="form-group">
            <label for="length">Länge (cm):</label>
            <input type="number" id="length" required>
        </div>
        <div class="form-group">
            <label for="width">Breite (cm):</label>
            <input type="number" id="width" required>
        </div>
        <div class="form-group">
            <label for="height">Höhe (cm):</label>
            <input type="number" id="height" required>
        </div>
        <div class="form-group">
            <label for="weight">Gewicht (kg):</label>
            <input type="number" id="weight" required>
        </div>
        <div class="form-group">
            <label for="from_country">Von (Land):</label>
            <select id="from_country" required>
                <option value="DE">Deutschland</option>
                <option value="AT">Österreich</option>
                <option value="CH">Schweiz</option>
            </select>
        </div>
        <div class="form-group">
            <label for="from_zip">Von (PLZ):</label>
            <input type="text" id="from_zip" required oninput="fetchCity('from')">
        </div>
        <div class="form-group">
            <label for="from_location">Von (Ort):</label>
            <input type="text" id="from_location" list="from_city_list">
            <datalist id="from_city_list"></datalist>
        </div>
        <div class="form-group">
            <label for="to_country">Nach (Land):</label>
            <select id="to_country" required>
                <option value="DE">Deutschland</option>
                <option value="AT">Österreich</option>
                <option value="CH">Schweiz</option>
            </select>
        </div>
        <div class="form-group">
            <label for="to_zip">Nach (PLZ):</label>
            <input type="text" id="to_zip" required oninput="fetchCity('to')">
        </div>
        <div class="form-group">
            <label for="to_location">Nach (Ort):</label>
            <input type="text" id="to_location" list="to_city_list">
            <datalist id="to_city_list"></datalist>
        </div>
        <button onclick="calculatePrice()">Preis und Laufzeit berechnen</button>
        <div class="result" id="result"></div>
    </div>

    <script>
        async function fetchCity(prefix) {
            const zip = document.getElementById(`${prefix}_zip`).value;
            if (zip.length < 5) return;

            try {
                const response = await fetch(`https://api.zippopotam.us/de/${zip}`);
                if (!response.ok) {
                    console.error(`Fehler beim Abrufen der Stadt für ${prefix} mit PLZ ${zip}`);
                    return;
                }

                const data = await response.json();
                const cityList = document.getElementById(`${prefix}_city_list`);
                cityList.innerHTML = '';

                data.places.forEach(place => {
                    const option = document.createElement('option');
                    option.value = place['place name'];
                    cityList.appendChild(option);

                    // Automatisch den ersten Ort ergänzen, wenn kein Ort manuell eingegeben wurde
                    if (prefix === 'from' && !document.getElementById('from_location').value) {
                        document.getElementById('from_location').value = place['place name'];
                    }
                    if (prefix === 'to' && !document.getElementById('to_location').value) {
                        document.getElementById('to_location').value = place['place name'];
                    }
                });
            } catch (error) {
                console.error('Fehler beim Abrufen der Stadt:', error);
            }
        }

        async function calculatePrice() {
            const length = document.getElementById('length').value;
            const width = document.getElementById('width').value;
            const height = document.getElementById('height').value;
            const weight = document.getElementById('weight').value;
            const from_country = document.getElementById('from_country').value;
            const from_zip = document.getElementById('from_zip').value;
            const from_location = document.getElementById('from_location').value;
            const to_country = document.getElementById('to_country').value;
            const to_zip = document.getElementById('to_zip').value;
            const to_location = document.getElementById('to_location').value;

            try {
                const response = await fetch('https://httpbin.org/post', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ length, width, height, weight, from_country, from_zip, from_location, to_country, to_zip, to_location })
                });

                const data = await response.json();
                console.log(data);

                // Simulating the best options for demo purposes
                const best_options = [
                    { provider: 'Anbieter 1', price: 100, duration: '2 Tage' },
                    { provider: 'Anbieter 2', price: 120, duration: '1 Tag' },
                    { provider: 'Anbieter 3', price: 90, duration: '3 Tage' }
                ];

                let resultHTML = '<h2>Beste Optionen</h2>';
                best_options.forEach(option => {
                    resultHTML += `
                        <p>Anbieter: ${option.provider}</p>
                        <p>Preis: ${option.price} €</p>
                        <p>Laufzeit: ${option.duration}</p>
                        <hr>
                    `;
                });
                document.getElementById('result').innerHTML = resultHTML;
            } catch (error) {
                console.error('Fehler beim Berechnen der Preise:', error);
                document.getElementById('result').innerHTML = '<p>Fehler beim Berechnen der Preise. Bitte versuchen Sie es später erneut.</p>';
            }
        }
    </script>
</body>
</html>
