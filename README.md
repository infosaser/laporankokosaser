<!DOCTYPE html>
<html lang="ms">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard Laporan Kokurikulum Sekolah (Live)</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- PapaParse untuk Pembacaan CSV Live dari Google Sheets -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
    .badge-daerah { background-color: #dbeafe; color: #1e40af; }
    .badge-negeri { background-color: #fef3c7; color: #92400e; }
    .badge-kebangsaan { background-color: #dcfce7; color: #166534; }
    .badge-antarabangsa { background-color: #fce7f3; color: #9d174d; }
    .badge-zon { background-color: #e0e7ff; color: #3730a3; }
  </style>
</head>
<body class="p-4 md:p-8">

  <!-- Header -->
  <header class="mb-8 bg-white p-6 rounded-2xl shadow-sm border border-slate-200 flex flex-col md:flex-row justify-between items-center gap-4">
    <div>
      <h1 class="text-2xl md:text-3xl font-bold text-slate-800 flex items-center gap-3">
        <i class="fa-solid fa-trophy text-amber-500"></i>
        Dashboard Laporan Kokurikulum (Live Data)
      </h1>
      <p class="text-slate-500 text-sm mt-1">Analisis Real-Time Penyertaan & Pencapaian Aktiviti Kokurikulum</p>
    </div>
    <div class="flex items-center gap-3">
      <span id="loadingStatus" class="text-xs text-indigo-600 font-semibold flex items-center gap-2">
        <i class="fa-solid fa-spinner fa-spin"></i> Memuatkan data...
      </span>
      <button onclick="fetchSheetData()" class="bg-indigo-600 hover:bg-indigo-700 text-white text-sm px-4 py-2 rounded-xl transition shadow flex items-center gap-2">
        <i class="fa-solid fa-rotate-right"></i> Kemas Kini Data Live
      </button>
    </div>
  </header>

  <!-- Summary Cards (5 Kad Ringkasan) -->
  <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-5 gap-4 mb-8">
    <!-- Card Total -->
    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
      <div class="text-slate-500 text-xs font-semibold uppercase tracking-wider">Jumlah Pertandingan</div>
      <div id="card-total" class="text-3xl font-bold text-slate-800 mt-2">0</div>
      <div class="text-xs text-slate-400 mt-1">Aktiviti Didaftarkan</div>
    </div>

    <!-- Card Antarabangsa (Ditambah Baru) -->
    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
      <div class="text-slate-500 text-xs font-semibold uppercase tracking-wider">Antarabangsa</div>
      <div id="card-antarabangsa" class="text-3xl font-bold text-pink-600 mt-2">0</div>
      <div id="card-antarabangsa-pct" class="text-xs text-pink-500 mt-1">0% daripada keseluruhan</div>
    </div>

    <!-- Card Kebangsaan -->
    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
      <div class="text-slate-500 text-xs font-semibold uppercase tracking-wider">Kebangsaan</div>
      <div id="card-kebangsaan" class="text-3xl font-bold text-emerald-600 mt-2">0</div>
      <div id="card-kebangsaan-pct" class="text-xs text-emerald-500 mt-1">0% daripada keseluruhan</div>
    </div>

    <!-- Card Negeri -->
    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
      <div class="text-slate-500 text-xs font-semibold uppercase tracking-wider">Negeri</div>
      <div id="card-negeri" class="text-3xl font-bold text-amber-600 mt-2">0</div>
      <div id="card-negeri-pct" class="text-xs text-amber-500 mt-1">0% daripada keseluruhan</div>
    </div>

    <!-- Card Daerah -->
    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
      <div class="text-slate-500 text-xs font-semibold uppercase tracking-wider">Daerah</div>
      <div id="card-daerah" class="text-3xl font-bold text-blue-600 mt-2">0</div>
      <div id="card-daerah-pct" class="text-xs text-blue-500 mt-1">0% daripada keseluruhan</div>
    </div>
  </div>

  <!-- Charts Section -->
  <div class="grid grid-cols-1 md:grid-cols-2 gap-8 mb-8">
    <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
      <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
        <i class="fa-solid fa-chart-pie text-indigo-500"></i>
        Peratus Penyertaan Mengikut Peringkat
      </h2>
      <div class="relative h-64 flex justify-center items-center">
        <canvas id="peringkatChart"></canvas>
      </div>
    </div>

    <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
      <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
        <i class="fa-solid fa-chart-column text-indigo-500"></i>
        Penyertaan Mengikut Unit Kokurikulum
      </h2>
      <div class="relative h-64">
        <canvas id="unitChart"></canvas>
      </div>
    </div>
  </div>

  <!-- Filters Section -->
  <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 mb-8">
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <div>
        <label class="block text-xs font-semibold text-slate-600 uppercase mb-2">Carian Pertandingan / Murid / Guru</label>
        <div class="relative">
          <i class="fa-solid fa-magnifying-glass absolute left-3 top-3 text-slate-400"></i>
          <input type="text" id="searchInput" oninput="applyFilters()" placeholder="Cari aktiviti, nama murid, guru..." 
                 class="w-full pl-9 pr-4 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
        </div>
      </div>

      <div>
        <label class="block text-xs font-semibold text-slate-600 uppercase mb-2">Peringkat Pertandingan</label>
        <select id="filterPeringkat" onchange="applyFilters()" class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white">
          <option value="Semua">Semua Peringkat</option>
          <option value="DAERAH">Daerah</option>
          <option value="NEGERI">Negeri</option>
          <option value="KEBANGSAAN">Kebangsaan</option>
          <option value="ANTARABANGSA">Antarabangsa</option>
          <option value="ZON">Zon</option>
        </select>
      </div>

      <div>
        <label class="block text-xs font-semibold text-slate-600 uppercase mb-2">Unit Kokurikulum</label>
        <select id="filterUnit" onchange="applyFilters()" class="w-full px-3 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white">
          <option value="Semua">Semua Unit</option>
          <option value="SUKAN PERMAINAN">Sukan Permainan</option>
          <option value="KELAB / PERSATUAN">Kelab / Persatuan</option>
          <option value="PASUKAN PAKAIAN BERUNIFORM">Pasukan Pakaian Beruniform</option>
          <option value="LAIN-LAIN">Lain-lain</option>
        </select>
      </div>
    </div>
  </div>

  <!-- Data List / Cards -->
  <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
    <div class="p-6 border-b border-slate-100 flex justify-between items-center">
      <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
        <i class="fa-solid fa-list-check text-indigo-500"></i>
        Senarai Pertandingan & Laporan
      </h2>
      <span id="recordCount" class="text-xs bg-slate-100 text-slate-600 font-semibold px-3 py-1 rounded-full">0 Rekod Dijumpai</span>
    </div>

    <div id="cardsContainer" class="p-6 grid grid-cols-1 md:grid-cols-2 gap-6">
      <!-- Kad dinamik -->
    </div>
  </div>

  <script>
    const SHEET_CSV_URL = 'https://docs.google.com/spreadsheets/d/11xOOewueDHz0c8pM2kVKy1RXAAs_6Y1QgYeaDYvdyoM/gviz/tq?tqx=out:csv';

    let allData = [];
    let peringkatChartInstance = null;
    let unitChartInstance = null;

    function fetchSheetData() {
      const statusEl = document.getElementById('loadingStatus');
      statusEl.innerHTML = `<i class="fa-solid fa-spinner fa-spin"></i> Memuatkan data...`;
      statusEl.className = "text-xs text-indigo-600 font-semibold flex items-center gap-2";

      Papa.parse(SHEET_CSV_URL, {
        download: true,
        header: true,
        skipEmptyLines: true,
        complete: function(results) {
          allData = transformRawData(results.data);
          renderDashboard(allData);

          statusEl.innerHTML = `<i class="fa-solid fa-check-circle text-emerald-500"></i> Berjaya Muat ${allData.length} Rekod`;
          statusEl.className = "text-xs text-emerald-600 font-semibold flex items-center gap-2";
        },
        error: function(err) {
          console.error("Gagal membaca Google Sheet CSV:", err);
          statusEl.innerHTML = `<i class="fa-solid fa-triangle-exclamation text-rose-500"></i> Gagal Muat Data`;
          statusEl.className = "text-xs text-rose-600 font-semibold flex items-center gap-2";
        }
      });
    }

    function transformRawData(rows) {
      return rows.map(row => {
        const peserta = [];
        for (let i = 1; i <= 7; i++) {
          const namaKey = `NAMA PESERTA ${i}`;
          const kelasKey = `KELAS PESERTA ${i}`;
          if (row[namaKey] && row[namaKey].trim() !== '') {
            peserta.push({
              nama: row[namaKey].trim(),
              kelas: row[kelasKey] ? row[kelasKey].trim() : ''
            });
          }
        }

        const guruUtama = row['NAMA GURU PENGIRING 1'] || '';
        const guru2 = row['NAMA GURU PENGIRING 2'] || '';
        const senaraiGuru = [guruUtama, guru2].filter(g => g.trim() !== '').join(', ');

        return {
          tarikh: row['TARIKH PERTANDINGAN / KEJOHANAN'] || row['Timestamp'] || '',
          unit: (row['UNIT KOKURIKULUM'] || 'LAIN-LAIN').trim(),
          pertandingan: row['NAMA PERTANDINGAN / PROGRAM'] || 'Pertandingan Tanpa Nama',
          peringkat: (row['PERINGKAT'] || 'DAERAH').trim(),
          tempat: row['TEMPAT PERTANDINGAN / KEJOHANAN'] || '-',
          pencapaian: row['KEPUTUSAN / PENCAPAIAN'] || 'PENYERTAAN',
          guru: senaraiGuru || '-',
          peserta: peserta,
          docUrl: row['Merged Doc URL - LAPORAN GURU PENGIRING AKTIVITI KOKURIKULUM 2026'] || row['SENARAI PESERTA  : (JIKA RAMAI)'] || ''
        };
      });
    }

    function renderDashboard(data) {
      updateCards(data);
      renderCharts(data);
      renderCardsList(data);
    }

    function updateCards(data) {
      const total = data.length;
      document.getElementById('card-total').innerText = total;

      const countAntarabangsa = data.filter(d => d.peringkat.toUpperCase().includes('ANTARABANGSA')).length;
      const countKebangsaan = data.filter(d => d.peringkat.toUpperCase().includes('KEBANGSAAN')).length;
      const countNegeri = data.filter(d => d.peringkat.toUpperCase().includes('NEGERI')).length;
      const countDaerah = data.filter(d => d.peringkat.toUpperCase().includes('DAERAH')).length;

      // Kemas kini Kad Antarabangsa dan %
      document.getElementById('card-antarabangsa').innerText = countAntarabangsa;
      document.getElementById('card-antarabangsa-pct').innerText = total > 0 ? ((countAntarabangsa/total)*100).toFixed(1) + "% penyertaan" : "0%";

      // Kemas kini Kad Kebangsaan dan %
      document.getElementById('card-kebangsaan').innerText = countKebangsaan;
      document.getElementById('card-kebangsaan-pct').innerText = total > 0 ? ((countKebangsaan/total)*100).toFixed(1) + "% penyertaan" : "0%";

      // Kemas kini Kad Negeri dan %
      document.getElementById('card-negeri').innerText = countNegeri;
      document.getElementById('card-negeri-pct').innerText = total > 0 ? ((countNegeri/total)*100).toFixed(1) + "% penyertaan" : "0%";

      // Kemas kini Kad Daerah dan %
      document.getElementById('card-daerah').innerText = countDaerah;
      document.getElementById('card-daerah-pct').innerText = total > 0 ? ((countDaerah/total)*100).toFixed(1) + "% penyertaan" : "0%";
    }

    function renderCharts(data) {
      const peringkatCounts = { DAERAH: 0, NEGERI: 0, KEBANGSAAN: 0, ANTARABANGSA: 0, ZON: 0 };

      data.forEach(d => {
        const p = d.peringkat.toUpperCase();
        if (p.includes('DAERAH')) peringkatCounts.DAERAH++;
        else if (p.includes('NEGERI')) peringkatCounts.NEGERI++;
        else if (p.includes('KEBANGSAAN')) peringkatCounts.KEBANGSAAN++;
        else if (p.includes('ANTARABANGSA')) peringkatCounts.ANTARABANGSA++;
        else if (p.includes('ZON')) peringkatCounts.ZON++;
      });

      const ctxPeringkat = document.getElementById('peringkatChart').getContext('2d');
      if (peringkatChartInstance) peringkatChartInstance.destroy();

      peringkatChartInstance = new Chart(ctxPeringkat, {
        type: 'doughnut',
        data: {
          labels: ['Daerah', 'Negeri', 'Kebangsaan', 'Antarabangsa', 'Zon'],
          datasets: [{
            data: Object.values(peringkatCounts),
            backgroundColor: ['#3b82f6', '#f59e0b', '#10b981', '#ec4899', '#6366f1']
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { position: 'right' } }
        }
      });

      const unitCounts = {};
      data.forEach(d => {
        const u = d.unit || 'LAIN-LAIN';
        unitCounts[u] = (unitCounts[u] || 0) + 1;
      });

      const ctxUnit = document.getElementById('unitChart').getContext('2d');
      if (unitChartInstance) unitChartInstance.destroy();

      unitChartInstance = new Chart(ctxUnit, {
        type: 'bar',
        data: {
          labels: Object.keys(unitCounts),
          datasets: [{
            label: 'Bilangan Aktiviti',
            data: Object.values(unitCounts),
            backgroundColor: '#6366f1'
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
        }
      });
    }

    function getBadgeClass(peringkat) {
      const p = peringkat.toUpperCase();
      if (p.includes('DAERAH')) return 'badge-daerah';
      if (p.includes('NEGERI')) return 'badge-negeri';
      if (p.includes('KEBANGSAAN')) return 'badge-kebangsaan';
      if (p.includes('ANTARABANGSA')) return 'badge-antarabangsa';
      return 'badge-zon';
    }

    function renderCardsList(data) {
      const container = document.getElementById('cardsContainer');
      document.getElementById('recordCount').innerText = `${data.length} Rekod Dijumpai`;
      container.innerHTML = '';

      if (data.length === 0) {
        container.innerHTML = `<div class="col-span-2 text-center py-12 text-slate-400">Tiada rekod pertandingan dijumpai mengikut carian/penapis ini.</div>`;
        return;
      }

      data.forEach(item => {
        let pesertaHtml = '';
        if (item.peserta && item.peserta.length > 0) {
          pesertaHtml = `
            <div class="mt-3 bg-slate-50 p-3 rounded-xl border border-slate-100">
              <div class="text-xs font-semibold text-slate-500 mb-1"><i class="fa-solid fa-users text-indigo-500 mr-1"></i> Peserta (${item.peserta.length} Orang):</div>
              <ul class="text-xs text-slate-700 space-y-1">
                ${item.peserta.map(p => `
                  <li class="flex justify-between border-b border-slate-200/60 pb-1 last:border-none">
                    <span>${p.nama}</span>
                    <span class="font-medium text-indigo-600">${p.kelas}</span>
                  </li>
                `).join('')}
              </ul>
            </div>
          `;
        } else {
          pesertaHtml = `
            <div class="mt-3 bg-slate-50 p-3 rounded-xl border border-slate-100 text-xs text-slate-500 italic">
              <i class="fa-solid fa-info-circle text-amber-500 mr-1"></i> Senarai peserta ramai/terperinci dalam dokumen laporan.
            </div>
          `;
        }

        const card = document.createElement('div');
        card.className = "bg-white p-5 rounded-xl border border-slate-200 hover:shadow-md transition flex flex-col justify-between";
        card.innerHTML = `
          <div>
            <div class="flex items-center justify-between gap-2 mb-2">
              <span class="text-xs font-bold px-2.5 py-1 rounded-full ${getBadgeClass(item.peringkat)}">${item.peringkat}</span>
              <span class="text-xs text-slate-400 font-medium">${item.tarikh}</span>
            </div>
            <h3 class="font-bold text-slate-800 text-base mb-1">${item.pertandingan}</h3>
            <p class="text-xs text-indigo-600 font-medium mb-3">${item.unit}</p>
            
            <div class="text-xs text-slate-600 space-y-1.5 mb-2">
              <div><i class="fa-solid fa-location-dot w-4 text-slate-400"></i> ${item.tempat}</div>
              <div><i class="fa-solid fa-award w-4 text-amber-500"></i> Pencapaian: <span class="font-bold text-slate-800">${item.pencapaian}</span></div>
              <div><i class="fa-solid fa-user-tie w-4 text-slate-400"></i> Guru: ${item.guru}</div>
            </div>

            ${pesertaHtml}
          </div>

          <div class="mt-4 pt-3 border-t border-slate-100 flex justify-end">
            ${item.docUrl ? `
              <a href="${item.docUrl}" target="_blank" class="text-xs text-indigo-600 hover:text-indigo-800 font-semibold flex items-center gap-1.5 bg-indigo-50 hover:bg-indigo-100 px-3 py-1.5 rounded-lg transition">
                <i class="fa-solid fa-file-pdf"></i> Lihat Laporan Rasmi (Google Form)
              </a>
            ` : ''}
          </div>
        `;
        container.appendChild(card);
      });
    }

    function applyFilters() {
      const search = document.getElementById('searchInput').value.toLowerCase();
      const peringkat = document.getElementById('filterPeringkat').value;
      const unit = document.getElementById('filterUnit').value;

      const filtered = allData.filter(d => {
        const matchesSearch = d.pertandingan.toLowerCase().includes(search) ||
                              d.guru.toLowerCase().includes(search) ||
                              d.pencapaian.toLowerCase().includes(search) ||
                              d.peserta.some(p => p.nama.toLowerCase().includes(search));

        const matchesPeringkat = (peringkat === "Semua") || d.peringkat.toUpperCase().includes(peringkat);
        const matchesUnit = (unit === "Semua") || d.unit.toUpperCase().includes(unit);

        return matchesSearch && matchesPeringkat && matchesUnit;
      });

      renderDashboard(filtered);
    }

    window.onload = function() {
      fetchSheetData();
    };
  </script>
</body>
</html>
