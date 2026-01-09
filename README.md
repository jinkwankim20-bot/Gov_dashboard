<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>서부안전팀 정압기 통합 대시보드</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.8/dist/web/static/pretendard.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha2d-p4z3r8gq333+7c3s2c4v4q0d1d2s3d4t5u6w7x8y9z0a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z" crossorigin=""/>
    
    <style>
        body {
            font-family: 'Pretendard', sans-serif;
            background-color: #f1f5f9;
            color: #1e293b; /* Slate-800 for better contrast */
        }

        /* Marker Icon Style - 크기 약간 확대 */
        .regulator-icon {
            text-align: center;
            line-height: 36px; /* Height와 맞춤 */
            font-weight: 800;
            font-size: 14px;
            color: white;
            border-radius: 50%;
            border: 3px solid #ffffff;
            width: 40px; /* 기존 32px -> 40px */
            height: 40px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.4);
            transition: all 0.2s ease;
        }
        .regulator-icon:hover {
            transform: scale(1.15);
            z-index: 999 !important;
        }

        /* Custom Scrollbar */
        .custom-scroll::-webkit-scrollbar {
            width: 8px;
        }
        .custom-scroll::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        .custom-scroll::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        .custom-scroll::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }

        select {
            background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3e%3cpath stroke='%2364748b' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3e%3c/svg%3e");
            background-position: right 0.75rem center;
            background-repeat: no-repeat;
            background-size: 1.25em 1.25em;
            padding-right: 2.5rem;
            -webkit-appearance: none;
            appearance: none;
        }
    </style>
</head>

<body class="flex flex-col h-screen lg:overflow-hidden overflow-auto bg-slate-100">

    <header class="bg-slate-900 text-white shadow-lg z-30 flex-shrink-0 h-16 flex items-center justify-center border-b border-slate-700">
        <div class="w-full px-6 flex items-center justify-between">
            <h1 class="text-lg lg:text-xl font-bold tracking-tight flex items-center gap-3">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="w-6 h-6 text-teal-400">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 3v11.25A2.25 2.25 0 0 0 6 16.5h2.25M3.75 3h-1.5m1.5 0h16.5m0 0h1.5m-1.5 0v11.25A2.25 2.25 0 0 1 18 16.5h-2.25m-7.5 0h7.5m-7.5 0-1 3m8.5-3 1 3m0 0 .5 1.5m-.5-1.5h-9.5m0 0-.5 1.5M9 11.25v1.5M12 9v3.75m3-6v6" />
                </svg>
                서부안전팀 정압기 현황
            </h1>
            
            <div class="flex items-center gap-4">
                <label for="csvFileInput" class="cursor-pointer bg-teal-600 hover:bg-teal-500 text-white px-4 py-2 rounded-lg text-sm font-bold shadow transition-colors flex items-center gap-2">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="w-5 h-5">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5m-13.5-9L12 3m0 0 4.5 4.5M12 3v13.5" />
                    </svg>
                    CSV 업로드
                </label>
                <input type="file" id="csvFileInput" accept=".csv" class="hidden" onchange="handleFileSelect(event)">
                <span id="fileStatus" class="text-sm text-slate-400 hidden lg:block">파일 없음</span>
            </div>
        </div>
        <div id="errorMsg" class="absolute top-16 w-full bg-rose-600 text-white text-sm font-bold text-center py-2 hidden shadow-md z-40"></div>
    </header>

    <main class="flex-grow flex flex-col lg:flex-row min-h-0 relative">
        
        <div class="relative w-full lg:flex-1 h-[40vh] lg:h-full z-0 order-1 border-r border-slate-300">
            <div id="map" class="w-full h-full bg-slate-200"></div>
            <div class="absolute top-5 left-5 z-[400] bg-white/95 backdrop-blur px-4 py-2 rounded-lg shadow-lg border border-slate-200">
                <h2 class="text-sm font-bold text-slate-800 flex items-center gap-2">
                    <span class="relative flex h-3 w-3">
                      <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-teal-400 opacity-75"></span>
                      <span class="relative inline-flex rounded-full h-3 w-3 bg-teal-500"></span>
                    </span>
                    GIS 위치 모니터링
                </h2>
            </div>
        </div>

        <div class="w-full lg:w-[420px] h-full flex flex-col bg-slate-50 z-10 order-2 shadow-xl">
            
            <div class="p-5 bg-white border-b border-slate-200 shadow-sm flex-shrink-0">
                <div class="flex items-center justify-between mb-3">
                    <h3 class="text-base font-bold text-slate-800 flex items-center gap-2">
                        <svg class="w-5 h-5 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z"></path></svg>
                        검색 필터
                    </h3>
                    <button onclick="resetFilters()" class="text-sm text-slate-500 hover:text-teal-600 font-medium underline underline-offset-2">초기화</button>
                </div>
                <div class="space-y-3">
                    <div class="grid grid-cols-2 gap-3">
                        <select id="loopFilter" onchange="applyFilters()" class="w-full bg-slate-50 border border-slate-300 text-slate-800 text-sm font-medium rounded-lg focus:ring-2 focus:ring-teal-500 block p-2.5 outline-none" disabled>
                            <option value="all">LOOP 전체</option>
                        </select>
                        <select id="sectionFilter" onchange="applyFilters()" class="w-full bg-slate-50 border border-slate-300 text-slate-800 text-sm font-medium rounded-lg focus:ring-2 focus:ring-teal-500 block p-2.5 outline-none" disabled>
                            <option value="all">구간 전체</option>
                        </select>
                    </div>
                    <select id="modelFilter" onchange="applyFilters()" class="w-full bg-slate-50 border border-slate-300 text-slate-800 text-sm font-medium rounded-lg focus:ring-2 focus:ring-teal-500 block p-2.5 outline-none" disabled>
                        <option value="all">모델 전체</option>
                    </select>
                </div>
            </div>

            <div class="px-5 py-3 bg-slate-100 border-b border-slate-200 flex justify-between items-center flex-shrink-0">
                <span class="text-sm font-bold text-slate-600">조회 결과</span>
                <span id="list-count" class="bg-teal-600 text-white text-sm font-bold px-3 py-1 rounded-full shadow-sm">0개</span>
            </div>
            
            <div class="flex-1 overflow-y-auto custom-scroll bg-slate-100 p-4">
                <div id="data-list-container" class="min-h-full">
                    <div id="regulator-cards" class="flex flex-col gap-4">
                        <div class="flex flex-col items-center justify-center py-20 text-slate-400">
                            <p class="text-base font-medium">상단 'CSV 업로드' 버튼을 눌러<br>데이터를 불러오세요.</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </main>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha2d-p4z3r8gq333+7c3s2c4v4q0d1d2s3d4t5u6w7x8y9z0a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z" crossorigin=""></script>

    <script>
        // --- Configuration ---
        let regulators = [];
        let map = null;
        let markers = L.layerGroup(); 
        
        const LOOP_COLORS = {
            '1': '#3b82f6', '2': '#10b981', '3': '#f59e0b', '4': '#8b5cf6', 
            '5': '#ef4444', '6': '#06b6d4', '7': '#ec4899', '8': '#f97316', 
            '9': '#6366f1', '10': '#14b8a6', 'default': '#64748b'
        };

        const icons = {
            pressure: `<svg class="w-4 h-4 text-slate-400 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>`,
            valve: `<svg class="w-4 h-4 text-slate-400 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>`,
            user: `<svg class="w-4 h-4 text-slate-400 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>`,
            calendar: `<svg class="w-4 h-4 text-slate-400 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>`
        };

        // --- Logic ---
        function parseCSV(csvString) {
            const lines = csvString.trim().split('\n').filter(line => line.trim() !== '');
            if (lines.length < 2) throw new Error("데이터 형식이 올바르지 않습니다.");
            const headers = lines[0].split(',').map(h => h.trim());
            if (!headers.includes('Lon') || !headers.includes('Lat')) throw new Error("'Lon', 'Lat' 컬럼 필요");

            return lines.slice(1).map(line => {
                const values = line.split(',');
                let obj = {};
                headers.forEach((header, i) => {
                    obj[header.trim()] = values[i] ? values[i].trim() : '';
                });
                return obj;
            });
        }

        function determineInspectionStatus(dateStr) {
            const currentYear = new Date().getFullYear();
            let year = parseInt(dateStr.substring(0, 4), 10);
            if (isNaN(year)) year = currentYear;
            const isMatch = (currentYear % 2) === (year % 2);
            // 가독성을 위해 배경색을 좀 더 진하게 수정
            return isMatch 
                ? { text: '분해점검', class: 'bg-rose-100 text-rose-700 border border-rose-200' }
                : { text: '필터점검', class: 'bg-emerald-100 text-emerald-700 border border-emerald-200' };
        }

        function handleFileSelect(event) {
            const file = event.target.files[0];
            const status = document.getElementById('fileStatus');
            const err = document.getElementById('errorMsg');
            
            err.classList.add('hidden');
            if (!file) return;

            status.textContent = `로드 중...`;
            const reader = new FileReader();
            reader.onload = (e) => {
                try {
                    initDashboard(e.target.result);
                    status.textContent = `${file.name}`;
                    status.className = "text-sm text-teal-600 font-bold truncate max-w-[150px] lg:block hidden";
                } catch (error) {
                    err.textContent = `오류: ${error.message}`;
                    err.classList.remove('hidden');
                    status.textContent = '실패';
                }
            };
            reader.readAsText(file, 'euc-kr');
        }

        // --- UI Generation (Improved) ---
        function createRegulatorCard(regulator, loopColor) {
            const card = document.createElement('div');
            // 패딩(p-5)을 늘리고 그림자를 강화하여 카드 구분감 향상
            card.className = 'bg-white rounded-xl shadow-sm border border-slate-200 hover:border-teal-500 hover:shadow-md transition-all cursor-pointer group flex flex-col p-5';
            
            const isPriority = regulator.isPriority;
            const titleClass = isPriority ? 'text-rose-600' : 'text-slate-900'; // 제목 색상 진하게
            
            card.innerHTML = `
                <div class="flex items-start justify-between mb-4">
                    <div class="flex items-center gap-3">
                        <span class="flex-shrink-0 flex items-center justify-center w-8 h-8 rounded-full text-white text-sm font-bold shadow-sm" style="background-color: ${loopColor};">
                            ${regulator.LOOP === 'default' ? '-' : regulator.LOOP}
                        </span>
                        <div>
                            <h4 class="text-base font-bold ${titleClass} leading-tight">${regulator['정압기명']}</h4>
                            <p class="text-sm text-slate-500 mt-0.5">${regulator['모델명'] || '모델 미정'}</p>
                        </div>
                    </div>
                    <div class="flex flex-col items-end gap-1.5 flex-shrink-0">
                        ${isPriority ? '<span class="text-xs bg-rose-50 text-rose-600 px-2 py-1 rounded font-bold border border-rose-100">우선방출</span>' : ''}
                        <span class="text-xs px-2 py-1 rounded font-bold whitespace-nowrap ${regulator.inspection.class}">${regulator.inspection.text}</span>
                    </div>
                </div>
                
                <div class="bg-slate-50 rounded-lg border border-slate-200 p-3 grid grid-cols-2 gap-4 mb-3">
                    <div class="flex flex-col">
                        <span class="text-xs font-medium text-slate-500 flex items-center mb-1">${icons.pressure} 입/출구 압력</span>
                        <span class="text-sm font-bold text-slate-800 font-mono tracking-tight">
                            ${regulator['입구압력(Mpa)']} <span class="text-slate-400">/</span> ${regulator['출구압력(kPa)']}
                        </span>
                    </div>
                    <div class="flex flex-col border-l border-slate-200 pl-4">
                        <span class="text-xs font-medium text-slate-500 flex items-center mb-1">${icons.valve} 개도율</span>
                        <span class="text-sm font-bold text-slate-800 font-mono tracking-tight">
                            ${regulator['센싱밸브 개도율'] || '-'}%
                        </span>
                    </div>
                </div>

                <div class="flex items-center justify-between text-sm text-slate-500 pt-2 border-t border-slate-100">
                    <div class="flex items-center gap-3">
                        <span class="flex items-center font-medium">${icons.user} ${regulator['담당자']}</span>
                        <span class="text-slate-300">|</span>
                        <span class="font-medium">${regulator['구간명']}</span>
                    </div>
                    <span class="flex items-center text-xs text-slate-400">${regulator['시공감리일자']}</span>
                </div>
            `;
            
            card.addEventListener('click', () => {
                if (map) {
                    map.setView([regulator.Lat, regulator.Lon], 16, { animate: true });
                    if(window.innerWidth < 1024) {
                        window.scrollTo({ top: 0, behavior: 'smooth' });
                    }
                }
            });

            return card;
        }

        function populateFilters() {
            const els = ['loopFilter', 'modelFilter', 'sectionFilter'].map(id => document.getElementById(id));
            els.forEach(el => el.disabled = false);

            const clean = (sel) => { while(sel.options.length > 1) sel.remove(1); };
            els.forEach(clean);

            const loops = [...new Set(regulators.map(r => r.LOOP))].filter(l => l !== 'default').sort((a,b)=>parseInt(a)-parseInt(b));
            const models = [...new Set(regulators.map(r => r['정압기모델']).filter(Boolean))].sort();
            const sections = [...new Set(regulators.map(r => r['구간명']).filter(Boolean))].sort();

            const elLoop = document.getElementById('loopFilter');
            const elSection = document.getElementById('sectionFilter');
            const elModel = document.getElementById('modelFilter');

            loops.forEach(v => elLoop.add(new Option(`LOOP ${v}`, v)));
            sections.forEach(v => elSection.add(new Option(v, v)));
            models.forEach(v => elModel.add(new Option(v, v)));
        }

        function renderData(data) {
            const container = document.getElementById('regulator-cards');
            const countEl = document.getElementById('list-count');
            
            container.innerHTML = '';
            markers.clearLayers();
            countEl.textContent = `${data.length}개`;

            if (data.length === 0) {
                container.innerHTML = `<div class="text-center py-10 text-slate-500 font-medium">검색 결과가 없습니다.</div>`;
                return;
            }

            const bounds = [];
            data.forEach(r => {
                const loop = r.LOOP || 'default';
                const color = LOOP_COLORS[loop] || LOOP_COLORS['default'];
                
                container.appendChild(createRegulatorCard(r, color));

                if (!isNaN(r.Lat) && !isNaN(r.Lon)) {
                    const icon = L.divIcon({
                        className: '',
                        html: `<div class="regulator-icon" style="background-color: ${color};">${loop}</div>`,
                        iconSize: [40, 40], // 아이콘 크기 확대
                        iconAnchor: [20, 20],
                        popupAnchor: [0, -15]
                    });
                    
                    // 지도 팝업 폰트도 키움
                    const popup = `
                        <div class="font-sans min-w-[200px]">
                            <div class="font-bold text-slate-900 mb-2 border-b pb-2 flex justify-between items-center text-base">
                                <span>${r['정압기명']}</span>
                                <span class="text-xs text-slate-500 font-normal bg-slate-100 px-2 py-0.5 rounded">${r['구간명']}</span>
                            </div>
                            <div class="text-sm text-slate-700 space-y-1.5">
                                <div class="flex justify-between border-b border-dashed border-slate-200 pb-1"><span>입구압력:</span> <span class="font-bold">${r['입구압력(Mpa)']}</span></div>
                                <div class="flex justify-between border-b border-dashed border-slate-200 pb-1"><span>출구압력:</span> <span class="font-bold">${r['출구압력(kPa)']}</span></div>
                                <div class="flex justify-between pt-1"><span>개도율:</span> <span class="font-bold text-teal-600">${r['센싱밸브 개도율']}%</span></div>
                            </div>
                        </div>
                    `;

                    const m = L.marker([r.Lat, r.Lon], { icon: icon }).bindPopup(popup);
                    markers.addLayer(m);
                    bounds.push([r.Lat, r.Lon]);
                }
            });

            if (bounds.length && map) {
                map.fitBounds(bounds, { padding: [50, 50], maxZoom: 15 });
            }
        }

        function applyFilters() {
            const vLoop = document.getElementById('loopFilter').value;
            const vSec = document.getElementById('sectionFilter').value;
            const vMod = document.getElementById('modelFilter').value;

            const res = regulators.filter(r => {
                const mLoop = vLoop === 'all' || (r.LOOP && r.LOOP.toString() === vLoop);
                const mSec = vSec === 'all' || r['구간명'] === vSec;
                const mMod = vMod === 'all' || r['정압기모델'] === vMod;
                return mLoop && mSec && mMod;
            });
            renderData(res);
        }

        function resetFilters() {
            document.getElementById('loopFilter').value = 'all';
            document.getElementById('sectionFilter').value = 'all';
            document.getElementById('modelFilter').value = 'all';
            applyFilters();
        }

        function initDashboard(csv) {
            const raw = parseCSV(csv);
            regulators = raw.map(r => {
                let l = r.LOOP ? r.LOOP.trim() : 'default';
                if (!isNaN(parseInt(l))) l = parseInt(l).toString();
                return {
                    ...r,
                    Lon: parseFloat(r.Lon),
                    Lat: parseFloat(r.Lat),
                    isPriority: r['비고'] && r['비고'].includes('우선'),
                    LOOP: l,
                    inspection: determineInspectionStatus(r['시공감리일자'])
                };
            });

            if (!map) initMap(); 
            populateFilters();
            renderData(regulators);
            resetFilters();
        }

        function initMap() {
            map = L.map('map', { zoomControl: false }).setView([36.77, 126.45], 11);
            L.control.zoom({ position: 'bottomright' }).addTo(map);
            L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
                attribution: '&copy; OpenStreetMap',
                subdomains: 'abcd',
                maxZoom: 20
            }).addTo(map);
            
            markers.addTo(map);
            new ResizeObserver(() => map.invalidateSize()).observe(document.getElementById('map'));
        }
        
        document.addEventListener('DOMContentLoaded', () => {
             if (!map) initMap();
        });
    </script>
</body>
</html>
