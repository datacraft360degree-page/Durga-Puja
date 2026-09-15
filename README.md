<html lang="bn">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Durga Puja Parikrama Dashboard & Planner</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- FontAwesome -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #fcf8f2; }
    .editable:hover { background-color: #fef3c7; cursor: pointer; border-radius: 4px; }
    .editable:focus { outline: 2px solid #d97706; background-color: #ffffff; padding: 2px 4px; }
  </style>
</head>
<body class="p-4 md:p-8 text-gray-800">

  <div class="max-w-7xl mx-auto space-y-8">
    
    <!-- Sync Status Banner -->
    <div id="sync-status" class="hidden text-xs text-center py-2 px-4 rounded-lg bg-amber-100 text-amber-800 font-semibold border border-amber-300">
      <i class="fa-solid fa-spinner fa-spin mr-2"></i> Syncing with Google Sheet...
    </div>

    <!-- Header -->
    <div class="flex flex-col md:flex-row justify-between items-center bg-gradient-to-r from-red-700 via-red-600 to-amber-600 text-white p-6 rounded-2xl shadow-lg">
      <div>
        <h1 class="text-3xl font-extrabold flex items-center gap-3">
          <i class="fa-solid fa-gopuram text-amber-300"></i> দুর্গোৎসব পরিক্রমা Planner & Dashboard
        </h1>
        <p class="text-red-100 text-sm mt-1">Plan, track, and monitor your Durga Puja pandal hopping with auto-numbered lists.</p>
      </div>
      <div class="mt-4 md:mt-0 flex gap-3">
        <button onclick="fetchDataFromSheet()" class="bg-amber-300 hover:bg-amber-400 text-red-950 font-bold px-4 py-2 rounded-xl shadow transition duration-200 flex items-center gap-2 text-sm">
          <i class="fa-solid fa-arrows-rotate"></i> Refresh Data
        </button>
        <button onclick="addNewRow()" class="bg-amber-400 hover:bg-amber-500 text-red-950 font-bold px-4 py-2 rounded-xl shadow transition duration-200 flex items-center gap-2">
          <i class="fa-solid fa-plus"></i> Add New Day
        </button>
        <button onclick="resetData()" class="bg-red-800 hover:bg-red-900 text-white font-semibold px-4 py-2 rounded-xl shadow transition duration-200 text-sm">
          <i class="fa-solid fa-rotate-left"></i> Reset
        </button>
      </div>
    </div>

    <!-- Dashboard Cards -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
      <div class="bg-white p-5 rounded-2xl shadow-sm border border-amber-100 flex items-center justify-between">
        <div>
          <p class="text-sm font-semibold text-gray-500">Total Planned Pandals</p>
          <p id="stat-total-pandals" class="text-3xl font-bold text-gray-800 mt-1">0</p>
        </div>
        <div class="p-4 bg-amber-100 text-amber-600 rounded-full">
          <i class="fa-solid fa-toran-gate text-2xl"></i>
        </div>
      </div>

      <div class="bg-white p-5 rounded-2xl shadow-sm border border-green-100 flex items-center justify-between">
        <div>
          <p class="text-sm font-semibold text-gray-500">Pandals Visited</p>
          <p id="stat-visited-pandals" class="text-3xl font-bold text-green-600 mt-1">0</p>
        </div>
        <div class="p-4 bg-green-100 text-green-600 rounded-full">
          <i class="fa-solid fa-circle-check text-2xl"></i>
        </div>
      </div>

      <div class="bg-white p-5 rounded-2xl shadow-sm border border-red-100 flex items-center justify-between">
        <div>
          <p class="text-sm font-semibold text-gray-500">Remaining Pandals</p>
          <p id="stat-pending-pandals" class="text-3xl font-bold text-red-600 mt-1">0</p>
        </div>
        <div class="p-4 bg-red-100 text-red-600 rounded-full">
          <i class="fa-solid fa-clock text-2xl"></i>
        </div>
      </div>

      <div class="bg-white p-5 rounded-2xl shadow-sm border border-blue-100 flex items-center justify-between">
        <div>
          <p class="text-sm font-semibold text-gray-500">Primary Mode</p>
          <p id="stat-primary-transport" class="text-2xl font-bold text-blue-600 mt-1">Bike</p>
        </div>
        <div class="p-4 bg-blue-100 text-blue-600 rounded-full">
          <i class="fa-solid fa-motorcycle text-2xl"></i>
        </div>
      </div>
    </div>

    <!-- Analytics Charts -->
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div class="lg:col-span-2 bg-white p-6 rounded-2xl shadow-sm border border-amber-100">
        <h2 class="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
          <i class="fa-solid fa-chart-column text-amber-600"></i> Pandals Count per Tithi / Day
        </h2>
        <div class="h-64">
          <canvas id="pandalChart"></canvas>
        </div>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-sm border border-amber-100 flex flex-col justify-between">
        <div>
          <h2 class="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
            <i class="fa-solid fa-chart-pie text-amber-600"></i> Progress Overview
          </h2>
          <div class="h-48 flex justify-center">
            <canvas id="progressChart"></canvas>
          </div>
        </div>
        <div class="text-center mt-4">
          <span id="completion-percentage" class="text-sm font-bold text-amber-700 bg-amber-50 px-3 py-1 rounded-full border border-amber-200">0% Completed</span>
        </div>
      </div>
    </div>

    <!-- Editable Planner Table -->
    <div class="bg-white rounded-2xl shadow-sm border border-amber-100 overflow-hidden">
      <div class="p-5 bg-amber-50 border-b border-amber-100 flex justify-between items-center">
        <h2 class="text-lg font-bold text-gray-800 flex items-center gap-2">
          <i class="fa-solid fa-list-check text-amber-600"></i> Parikrama Schedule & Auto-Numbered Pandals
        </h2>
        <span class="text-xs text-gray-500 italic"><i class="fa-solid fa-pen"></i> Click any text to edit inline. Changes sync automatically to Google Sheets.</span>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse">
          <thead>
            <tr class="bg-gray-50 text-gray-600 text-xs uppercase tracking-wider border-b border-gray-200">
              <th class="p-4 text-center">Status</th>
              <th class="p-4">Day</th>
              <th class="p-4">Date</th>
              <th class="p-4">তিথি (Tithi)</th>
              <th class="p-4 w-2/5">Auto-Numbered Pandals List</th>
              <th class="p-4">Via Mode</th>
              <th class="p-4 text-center">Actions</th>
            </tr>
          </thead>
          <tbody id="schedule-tbody" class="divide-y divide-gray-100 text-sm">
            <!-- Dynamic rows rendered here -->
          </tbody>
        </table>
      </div>
    </div>

  </div>

  <script>
    // Replace with your Google Apps Script Web App URL
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxZDXzhyyNP7n_u6GP_2OG945Qlgob8hYyBFmVgALKVzdQ0p_UI4a97Kzu8NQoU5pHh/exec";

    const initialData = [
      { id: 1, day: "Saturday", date: "10th Oct", tithi: "মহালয়া", pandals: ["Kumortuli visit(Morning Time)"], via: "Bike", completed: false },
      { id: 2, day: "Sunday", date: "11th Oct", tithi: "মহা প্রথমা", pandals: ["Santosh Mitra Square", "College Square", "Hedua Park", "Shree Bhumi"], via: "Bike", completed: false },
      { id: 3, day: "Monday", date: "12th Oct", tithi: "মহা দ্বিতীয়া", pandals: ["Tala Prottay", "Dakhsindari"], via: "Bike", completed: false },
      { id: 4, day: "Tuesday", date: "13th Oct", tithi: "মহা তৃতীয়া", pandals: ["Newtown Sarbojanin"], via: "Bike", completed: false },
      { id: 5, day: "Wednesday", date: "14th Oct", tithi: "মহা চতুর্থী", pandals: ["Chorbagan", "Jorasakho", "Maniktala", "Amra Sobai Club(Arjunpur)"], via: "Bike", completed: false },
      { id: 6, day: "Thursday", date: "15th Oct", tithi: "মহাপঞ্চমী", pandals: ["Dum Dum Park Sarbojanin", "Tarun Dal", "Tarun Sangha", "Bharat Chakra", "Dum Dum Yubok Brinda", "Dum Dum Park Milemise Club", "Dakhsinpara"], via: "Bike", completed: false },
      { id: 7, day: "Friday", date: "16th Oct", tithi: "মহাষষ্ঠী", pandals: ["Ultodanga College Puja", "Ultodanga Pallyshree", "Ultodanga Yubok Bridno", "Jagarana Puja", "Surir Bagan", "Kar Bagan", "kabiraj Bagan", "Telenga Bagan", "Ultodanga Bidhan Sangha", "Ultodanga Gold lane", "Autobindo Setu", "Gauri Bari"], via: "Bike", completed: false },
      { id: 8, day: "Saturday", date: "17th Oct", tithi: "মহাসপ্তমী", pandals: [], via: "Public Transport", completed: false },
      { id: 9, day: "Sunday", date: "18th Oct", tithi: "মহাষ্টমী", pandals: ["Nalin Sarkar Street", "Sikdar Bagan", "Hatibagan Sarbojanin", "Nabin Pally", "Kasi Bose lane", "Sovabazar Beniatola", "Arihitola Yubok Brinda", "Arihitola Sarbojanin", "Kumortuli Park", "Kumortuli Sarbojanin", "Sovabazar Sarbojanin", "Bagbazar Sarbojanin", "jagat Mukherjee Park"], via: "Bike", completed: false },
      { id: 10, day: "Monday", date: "19th Oct", tithi: "মহানবমী", pandals: [], via: "Bike", completed: false },
      { id: 11, day: "Tuesday", date: "20th Oct", tithi: "বিজয়া দশমী", pandals: ["Own Para Pandal"], via: "Bike", completed: false }
    ];

    let scheduleData = [];
    let pandalChart, progressChart;

    // Show Sync Status
    function showStatus(message, isError = false) {
      const banner = document.getElementById('sync-status');
      banner.className = `text-xs text-center py-2 px-4 rounded-lg font-semibold border transition ${
        isError ? 'bg-red-100 text-red-800 border-red-300' : 'bg-amber-100 text-amber-800 border-amber-300'
      }`;
      banner.innerHTML = message;
      banner.classList.remove('hidden');
    }

    function hideStatus() {
      document.getElementById('sync-status').classList.add('hidden');
    }

    // Fetch Data from Google Sheet
    async function fetchDataFromSheet() {
      showStatus('<i class="fa-solid fa-spinner fa-spin mr-2"></i> Loading data from Google Sheet...');
      try {
        const response = await fetch(SCRIPT_URL);
        const data = await response.json();
        
        if (data && data.length > 0) {
          scheduleData = data;
        } else {
          scheduleData = JSON.parse(JSON.stringify(initialData));
          await syncToSheet(); // Populate initial data if sheet is empty
        }
        renderTable();
        hideStatus();
      } catch (err) {
        console.error("Fetch Error:", err);
        showStatus("Failed to fetch from Google Sheet. Check SCRIPT_URL configuration.", true);
        scheduleData = JSON.parse(JSON.stringify(initialData));
        renderTable();
      }
    }

    // Automatic Sync to Google Sheet after any entry/change
    async function syncToSheet() {
      showStatus('<i class="fa-solid fa-arrows-rotate fa-spin mr-2"></i> Saving changes to Google Sheet...');
      updateDashboard();
      try {
        await fetch(SCRIPT_URL, {
          method: 'POST',
          mode: 'no-cors',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(scheduleData)
        });
        showStatus('<i class="fa-solid fa-circle-check text-green-600 mr-2"></i> Synced successfully!');
        setTimeout(hideStatus, 2000);
      } catch (err) {
        console.error("Sync Error:", err);
        showStatus("Failed to save changes to Google Sheet.", true);
      }
    }

    // Render Table Rows with Auto-Numbering
    function renderTable() {
      const tbody = document.getElementById('schedule-tbody');
      tbody.innerHTML = '';

      scheduleData.forEach((row, rowIndex) => {
        const tr = document.createElement('tr');
        tr.className = row.completed ? 'bg-green-50/50' : 'hover:bg-amber-50/30';

        let pandalsHTML = `<ol class="list-decimal list-inside space-y-1">`;
        if (!row.pandals || row.pandals.length === 0) {
          pandalsHTML += `<li class="text-gray-400 italic list-none">No pandals planned</li>`;
        } else {
          row.pandals.forEach((pandal, pIndex) => {
            pandalsHTML += `
              <li class="group flex items-center justify-between hover:bg-amber-100/60 p-1 rounded">
                <span>
                  <span class="font-semibold text-amber-900 mr-1">${pIndex + 1}.</span>
                  <span class="editable" contenteditable="true" onblur="updatePandal(${rowIndex}, ${pIndex}, this.innerText)">${pandal}</span>
                </span>
                <button onclick="removePandal(${rowIndex}, ${pIndex})" class="text-gray-300 hover:text-red-500 opacity-0 group-hover:opacity-100 transition px-1">
                  <i class="fa-solid fa-xmark text-xs"></i>
                </button>
              </li>
            `;
          });
        }
        pandalsHTML += `</ol>`;
        pandalsHTML += `
          <button onclick="addPandal(${rowIndex})" class="mt-2 text-xs text-amber-700 hover:text-amber-900 font-semibold flex items-center gap-1 bg-amber-100/80 hover:bg-amber-200 px-2 py-1 rounded transition">
            <i class="fa-solid fa-plus text-[10px]"></i> Add Pandal
          </button>
        `;

        tr.innerHTML = `
          <td class="p-4 text-center">
            <input type="checkbox" ${row.completed ? 'checked' : ''} onchange="toggleComplete(${rowIndex})" class="w-5 h-5 text-amber-600 rounded border-gray-300 focus:ring-amber-500 cursor-pointer">
          </td>
          <td class="p-4 font-medium editable" contenteditable="true" onblur="updateField(${rowIndex}, 'day', this.innerText)">${row.day}</td>
          <td class="p-4 text-gray-600 editable" contenteditable="true" onblur="updateField(${rowIndex}, 'date', this.innerText)">${row.date}</td>
          <td class="p-4 font-semibold text-amber-800 editable" contenteditable="true" onblur="updateField(${rowIndex}, 'tithi', this.innerText)">${row.tithi}</td>
          <td class="p-4 text-gray-700">${pandalsHTML}</td>
          <td class="p-4 editable" contenteditable="true" onblur="updateField(${rowIndex}, 'via', this.innerText)">
            <span class="px-2 py-1 bg-gray-100 border border-gray-200 rounded text-xs font-semibold">${row.via}</span>
          </td>
          <td class="p-4 text-center">
            <button onclick="deleteRow(${rowIndex})" class="text-red-400 hover:text-red-600 transition"><i class="fa-solid fa-trash-can"></i></button>
          </td>
        `;
        tbody.appendChild(tr);
      });

      updateDashboard();
    }

    // Handlers for Row Fields & Auto Numbered Pandals
    function updateField(index, field, value) {
      if (scheduleData[index][field] !== value.trim()) {
        scheduleData[index][field] = value.trim();
        syncToSheet();
      }
    }

    function updatePandal(rowIndex, pandalIndex, value) {
      if (value.trim() === "") {
        removePandal(rowIndex, pandalIndex);
      } else {
        scheduleData[rowIndex].pandals[pandalIndex] = value.trim();
        syncToSheet();
      }
    }

    function addPandal(rowIndex) {
      const name = prompt("Enter Pandal Name:");
      if (name && name.trim() !== "") {
        if (!scheduleData[rowIndex].pandals) scheduleData[rowIndex].pandals = [];
        scheduleData[rowIndex].pandals.push(name.trim());
        renderTable();
        syncToSheet();
      }
    }

    function removePandal(rowIndex, pandalIndex) {
      scheduleData[rowIndex].pandals.splice(pandalIndex, 1);
      renderTable();
      syncToSheet();
    }

    function toggleComplete(index) {
      scheduleData[index].completed = !scheduleData[index].completed;
      renderTable();
      syncToSheet();
    }

    function addNewRow() {
      const newEntry = {
        id: Date.now(),
        day: "New Day",
        date: "Date",
        tithi: "Tithi",
        pandals: ["New Pandal Location"],
        via: "Bike",
        completed: false
      };
      scheduleData.push(newEntry);
      renderTable();
      syncToSheet();
    }

    function deleteRow(index) {
      if (confirm("Are you sure you want to delete this day?")) {
        scheduleData.splice(index, 1);
        renderTable();
        syncToSheet();
      }
    }

    function resetData() {
      if (confirm("Reset schedule back to original auto-numbered template?")) {
        scheduleData = JSON.parse(JSON.stringify(initialData));
        renderTable();
        syncToSheet();
      }
    }

    // Dashboard & Analytics Updates
    function updateDashboard() {
      let totalPandals = 0;
      let visitedPandals = 0;
      const labels = [];
      const pandalCounts = [];

      scheduleData.forEach(row => {
        const count = row.pandals ? row.pandals.length : 0;
        totalPandals += count;
        if (row.completed) {
          visitedPandals += count;
        }

        labels.push(row.tithi || row.day);
        pandalCounts.push(count);
      });

      const pendingPandals = Math.max(0, totalPandals - visitedPandals);
      const completionRate = totalPandals > 0 ? Math.round((visitedPandals / totalPandals) * 100) : 0;

      document.getElementById('stat-total-pandals').innerText = totalPandals;
      document.getElementById('stat-visited-pandals').innerText = visitedPandals;
      document.getElementById('stat-pending-pandals').innerText = pendingPandals;
      document.getElementById('completion-percentage').innerText = `${completionRate}% Completed`;

      renderCharts(labels, pandalCounts, visitedPandals, pendingPandals);
    }

    function renderCharts(labels, pandalCounts, visited, pending) {
      const ctxBar = document.getElementById('pandalChart').getContext('2d');
      if (pandalChart) pandalChart.destroy();
      
      pandalChart = new Chart(ctxBar, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Pandals Count',
            data: pandalCounts,
            backgroundColor: '#d97706',
            borderRadius: 6
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { display: false } },
          scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
        }
      });

      const ctxPie = document.getElementById('progressChart').getContext('2d');
      if (progressChart) progressChart.destroy();

      progressChart = new Chart(ctxPie, {
        type: 'doughnut',
        data: {
          labels: ['Visited', 'Remaining'],
          datasets: [{
            data: [visited, pending],
            backgroundColor: ['#10b981', '#ef4444']
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { position: 'bottom' } }
        }
      });
    }

    // Initialize Page on Load
    window.onload = () => {
      fetchDataFromSheet();
    };
  </script>
</body>
</html>
