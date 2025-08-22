<script type="text/javascript">
        var gk_isXlsx = false;
        var gk_xlsxFileLookup = {};
        var gk_fileData = {};
        function filledCell(cell) {
          return cell !== '' && cell != null;
        }
        function loadFileData(filename) {
        if (gk_isXlsx && gk_xlsxFileLookup[filename]) {
            try {
                var workbook = XLSX.read(gk_fileData[filename], { type: 'base64' });
                var firstSheetName = workbook.SheetNames[0];
                var worksheet = workbook.Sheets[firstSheetName];

                // Convert sheet to JSON to filter blank rows
                var jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1, blankrows: false, defval: '' });
                // Filter out blank rows (rows where all cells are empty, null, or undefined)
                var filteredData = jsonData.filter(row => row.some(filledCell));

                // Heuristic to find the header row by ignoring rows with fewer filled cells than the next row
                var headerRowIndex = filteredData.findIndex((row, index) =>
                  row.filter(filledCell).length >= filteredData[index + 1]?.filter(filledCell).length
                );
                // Fallback
                if (headerRowIndex === -1 || headerRowIndex > 25) {
                  headerRowIndex = 0;
                }

                // Convert filtered JSON back to CSV
                var csv = XLSX.utils.aoa_to_sheet(filteredData.slice(headerRowIndex)); // Create a new sheet from filtered array of arrays
                csv = XLSX.utils.sheet_to_csv(csv, { header: 1 });
                return csv;
            } catch (e) {
                console.error(e);
                return "";
            }
        }
        return gk_fileData[filename] || "";
        }
        </script><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Car Damage Tracker</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            overflow-x: hidden;
        }
        h1 {
            text-align: center;
        }
        .container {
            position: relative;
            display: inline-block;
        }
        .main-content {
            max-width: 400px;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }
        .marker {
            position: absolute;
            width: 10px;
            height: 10px;
            background-color: red;
            border-radius: 50%;
            pointer-events: none;
        }
        #damageList {
            max-height: 400px;
            overflow-y: auto;
            width: 100%;
            border: 1px solid #ccc;
            padding: 10px;
            margin-top: 20px;
        }
        #damageList ul {
            list-style-type: none;
            padding: 0;
        }
        #damageList li {
            margin-bottom: 10px;
            border-bottom: 1px solid #eee;
            padding-bottom: 5px;
        }
        #damageModal {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 20px;
            border: 1px solid #000;
            z-index: 1000;
        }
        #modalBackdrop {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 999;
        }
    </style>
</head>
<body>
    <h1>Car Damage Tracker</h1>
    <div class="main-content">
        <div class="container">
            <img id="carImage" src="https://static.wixstatic.com/media/979173_907aaba065dc4728bf11df75c5581a3d~mv2.jpg/v1/fill/w_395,h_600,al_c,q_80,enc_avif,quality_auto/979173_907aaba065dc4728bf11df75c5581a3d~mv2.jpg" width="400" height="600">
        </div>
        <div id="damageList">
            <h3>Damage List</h3>
            <ul id="damageItems"></ul>
        </div>
    </div>
    <br>
    <button onclick="saveDamages()">Save Damages</button>
    <button onclick="loadDamages()">Load Previous Damages</button>
    <button onclick="clearDamages()">Clear All</button>

    <!-- Modal for entering damage details -->
    <div id="damageModal" style="display: none;">
        <h3>Enter Damage Details</h3>
        <label>Date: <input type="date" id="damageDate"></label><br>
        <label>Time: <input type="time" id="damageTime"></label><br>
        <label>Description: <textarea id="damageDescription" rows="4" cols="30"></textarea></label><br>
        <button onclick="submitDamageDetails()">Submit</button>
        <button onclick="closeModal()">Cancel</button>
    </div>
    <div id="modalBackdrop" style="display: none;"></div>

    <script>
        let damages = [];
        let currentClick = null;

        function init() {
            const img = document.getElementById('carImage');
            img.onclick = getClickPosition;
            loadDamages();
        }

        function getClickPosition(e) {
            const pos = getCoordinates(e);
            if (pos) {
                currentClick = pos;
                showModal();
            }
        }

        function getCoordinates(e) {
            let PosX = 0, PosY = 0;
            const img = document.getElementById('carImage');
            const ImgPos = findPosition(img);
            if (!e) e = window.event;
            if (e.pageX || e.pageY) {
                PosX = e.pageX;
                PosY = e.pageY;
            } else if (e.clientX || e.clientY) {
                PosX = e.clientX + document.body.scrollLeft + document.documentElement.scrollLeft;
                PosY = e.clientY + document.body.scrollTop + document.documentElement.scrollTop;
            }
            PosX -= ImgPos[0];
            PosY -= ImgPos[1];
            return {x: PosX, y: PosY};
        }

        function findPosition(oElement) {
            let posX = 0, posY = 0;
            for (; oElement; oElement = oElement.offsetParent) {
                posX += oElement.offsetLeft;
                posY += oElement.offsetTop;
            }
            return [posX, posY];
        }

        function addMarker(x, y) {
            const marker = document.createElement('div');
            marker.className = 'marker';
            marker.style.left = `${x - 5}px`;
            marker.style.top = `${y - 5}px`;
            document.querySelector('.container').appendChild(marker);
        }

        function showModal() {
            document.getElementById('damageModal').style.display = 'block';
            document.getElementById('modalBackdrop').style.display = 'block';
            document.getElementById('damageDate').value = '';
            document.getElementById('damageTime').value = '';
            document.getElementById('damageDescription').value = '';
        }

        function closeModal() {
            document.getElementById('damageModal').style.display = 'none';
            document.getElementById('modalBackdrop').style.display = 'none';
            currentClick = null;
        }

        function submitDamageDetails() {
            const date = document.getElementById('damageDate').value;
            const time = document.getElementById('damageTime').value;
            const description = document.getElementById('damageDescription').value;
            if (currentClick && date && time && description) {
                const damage = {
                    x: currentClick.x,
                    y: currentClick.y,
                    date: date,
                    time: time,
                    description: description
                };
                damages.push(damage);
                addMarker(currentClick.x, currentClick.y);
                updateDamageList();
                closeModal();
            } else {
                alert('Please fill in all fields.');
            }
        }

        function updateDamageList() {
            const list = document.getElementById('damageItems');
            list.innerHTML = '';
            damages.forEach((d, index) => {
                const li = document.createElement('li');
                li.innerHTML = `<strong>Damage ${index + 1}</strong>: (${d.x}, ${d.y})<br>Date: ${d.date}<br>Time: ${d.time}<br>Description: ${d.description}`;
                list.appendChild(li);
            });
        }

        function saveDamages() {
            localStorage.setItem('carDamages', JSON.stringify(damages));
            alert('Damages saved to local storage!');
        }

        function loadDamages() {
            const saved = localStorage.getItem('carDamages');
            if (saved) {
                damages = JSON.parse(saved);
                damages.forEach(d => addMarker(d.x, d.y));
                updateDamageList();
            }
        }

        function clearDamages() {
            damages = [];
            localStorage.removeItem('carDamages');
            document.querySelectorAll('.marker').forEach(m => m.remove());
            document.getElementById('damageItems').innerHTML = '';
            alert('All damages cleared!');
        }

        init();
    </script>
</body>
</html>
