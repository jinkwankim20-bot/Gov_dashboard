<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="mobile-web-app-capable" content="yes">
    <title>서부안전팀 정압기 통합 대시보드</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    
    <link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.8/dist/web/static/pretendard.css" />
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha2d-p4z3r8gq333+7c3s2c4v4q0d1d2s3d4t5u6w7x8y9z0a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z" crossorigin=""/>
    
    <style>
        /* 1. 기본 폰트 및 사이즈 강제 고정 (브라우저 차이 제거) */
        html {
            font-size: 16px; /* 기준 폰트 사이즈 고정 */
            -webkit-text-size-adjust: 100%; /* 모바일 폰트 자동 확대 방지 */
        }
        
        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, sans-serif;
            margin: 0;
            padding: 0;
            overflow: hidden; /* 전체 스크롤 막기 (앱처럼 동작) */
            background-color: #f1f5f9;
            color: #1e293b;
        }

        /* 2. 레이아웃 핵심 설정 */
        .app-container {
            display: flex;
            flex-direction: column;
            height: 100vh; /* 뷰포트 100% */
            height: 100dvh; /* 모바일 동적 뷰포트 대응 */
            width: 100vw;
        }

        .content-area {
            flex: 1;
            display: flex;
            overflow: hidden; /* 내부 스크롤을 위해 숨김 */
            position: relative;
        }

        /* 3. 지도 및 사이드바 스타일 */
        #map {
            width: 100%;
            height: 100%;
            background-color: #cbd5e1; /* 지도 로딩 전 배경색 */
            z-index: 1;
        }

        /* 사이드바: 데스크탑(고정), 모바일(반응형) */
        .sidebar {
            width: 420px;
            background: white;
            display: flex;
            flex-direction: column;
            border-left: 1px solid #e2e8f0;
            z-index: 10;
            box-shadow: -4px 0 15px rgba(0,0,0,0.05);
            flex-shrink: 0; /* 지도에 밀리지 않게 고정 */
        }

        /* 모바일 대응 미디어 쿼리 */
        @media (max-width: 1024px) {
            .content-area {
                flex-direction: column; /* 위아래 배치 */
            }
            .sidebar {
                width: 100%;
                height: 50%; /* 화면 절반 차지 */
                border-left: none;
                border-top: 1px solid #e2e8f0;
            }
            #map-wrapper {
                height: 50%; /* 지도 절반 */
            }
        }

        /* 4. 유틸리티 스타일 */
        .regulator-icon {
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 800;
            font-size: 14px;
            color: white;
            border-radius: 50%;
            border: 3px solid #ffffff;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.3);
            transition: transform 0.2s;
        }
        .regulator-icon:hover { transform: scale(1.15); z-index: 1000 !important; }

        .custom-scroll { overflow-y: auto; -webkit-overflow-scrolling: touch; }
        .custom-scroll::-webkit-scrollbar { width: 6px; }
        .custom-scroll::-webkit-scrollbar-track { background: transparent; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
        
        select {
            -webkit-appearance: none; appearance: none;
            background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3e%3cpath stroke='%2364748b' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3e%3c/svg%3e");
            background-position: right 0.75rem center; background-repeat: no-repeat; background-size: 1.25em 1.25em;
            padding-right: 2.5rem;
        }
    </style>
</head>
<body>

<div class="app-container">
    <header class="h-16 flex-none bg-slate-900 text-white flex items-center justify-between px-4 lg:px-6 shadow-md z-50">
        <div class="flex items-center gap-3">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="currentColor" class="w-6 h-6 text-teal-400">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 3v11.25A2.25 2.25 0 0 0 6 16.5h2.25M3.75 3h-1.5m1.5 0h16.5m0 0h1.5m-1.5 0v11.25A2.25 2.25 0 0 1 18 16.5h-2.25m-7.5 0h7.5m-7.5 0-1 3m8.5-3 1 3m0 0 .5 1.5m-.5-1.5h-9.5m0 0-.5 1.5M9 11.25v1.5M12 9v3.75m3-6v6" />
            </svg>
            <h1 class="text-lg lg:text-xl font-bold tracking-tight truncate">서부안전팀 통합 대시보드</h1>
        </div>
        <div class="flex items-center gap-3">
            <span id="fileStatus" class="hidden md:block text-sm text-slate-400 font-medium max-w-[150px] truncate">파일 없음</span>
            <label for="csvFileInput" class="cursor-pointer bg-teal-600 hover:bg-teal-500 text-white px-4 py-2 rounded-lg shadow text-sm font-bold flex items-center gap-2 transition-colors whitespace-nowrap">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" />
                </svg>
                <span class="hidden sm:inline">CSV 업로드</span>
                <span class="sm:hidden">업로드</span>
            </label>
            <input type="file" id="csvFileInput" accept=".csv" class="hidden" onchange="handleFileSelect(event)">
        </div>
    </header>

    <main class="content-area">
        
        <div id="map-wrapper" class="flex-1 relative w-full h-full">
            <div id="map"></div>
            
            <div class="absolute top-4 left-4 z-[400] bg-white/95 backdrop-blur px-3 py-2 rounded-lg shadow-lg border border-slate-200">
                <h2 class="text-sm font-extrabold text-slate-800 flex items-center gap-2">
                    <span class="relative flex h-3 w-3">
                        <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-teal-400 opacity-75"></span>
                        <span class="relative inline-flex rounded-full h-3 w-3 bg-teal-500"></span>
                    </span>
                    GIS 모니터링
                </h2>
            </div>

            <div id="errorMsg" class="absolute top-4 left-1/2 transform -translate-x-1/2 z-[1000] bg-rose-600 text-white px-5 py-2.5 rounded-lg shadow-xl font-bold text-sm hidden whitespace-nowrap"></div>
        </div>

        <aside class="sidebar">
            
            <div class="p-4 bg-slate-50 border-b border-slate-200 flex-shrink-0">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="text-base font-bold text-slate-800 flex items-center gap-2">
                        <svg class="w-5 h-5 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z"></path></svg>
                        검색 필터
                    </h3>
                    <button onclick="resetFilters()" class="text-sm text-slate-500 hover:text-teal-600 font-medium underline underline-offset-2">초기화</button>
                </div>
                
                <div class="space-y-2">
                    <div class="flex gap-2">
                        <select id="loopFilter" onchange="applyFilters()" class="flex-1 bg-white border border-slate-300 text-slate-900 text-sm rounded-lg focus:ring-2 focus:ring-teal-500 block p-2.5 shadow-sm font-medium" disabled>
                            <option value="all">LOOP</option>
                        </select>
                        <select id="sectionFilter" onchange="applyFilters()" class="flex-1 bg-white border border-slate-300 text-slate-900 text-sm rounded-lg focus:ring-2 focus:ring-teal-500 block p-2.5 shadow-sm font-medium" disabled>
                            <option value="all">구간명</option>
                        </select>
                    </div>
                    <select id="modelFilter" onchange="applyFilters()" class="w-full bg-white border border-slate-300 text-slate-900 text-sm rounded-lg focus:ring-2 focus:ring-teal-500 block p-2.5 shadow-sm font-medium" disabled>
                        <option value="all">모델 전체</option>
                    </select>
                </div>
            </div>

            <div class="px-4 py-2 bg-white border-b border-slate-200 flex justify-between items-center shadow-sm z-10 flex-shrink-0">
                <span class="text-sm font-bold text-slate-600">정압기 목록</span>
                <span id="list-count" class="bg-teal-100 text-teal-700 text-sm font-extrabold px-2.5 py-0.5 rounded-full border border-teal-200">0건</span>
            </div>

            <div class="flex-1 custom-scroll bg-slate-100 p-3">
                <div id="regulator-cards" class="flex flex-col gap-3 pb-8">
                    <div class="flex flex-col items-center justify-center py-16 text-slate-400">
                        <svg class="w-12 h-12 mb-3 opacity-30" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"></path></svg>
                        <p class="text-sm font-medium">데이터 파일(CSV)을 업로드하세요.</p>
                    </div>
                </div>
            </div>
        </aside>
    </main>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha2d-p4z3r8gq333+7c3s2c4v4q0d1d2s3d4t5u6w7x8y9z0a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z" crossorigin=""></script>
<script>
    // --- Global Config ---
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
        user: `<svg class="w-4 h-4 text-slate-400 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>`
    };

    // --- CSV Parsing ---
    function parseCSV(csvString) {
        const lines = csvString.trim().split('\n').filter(line => line.trim() !== '');
        if (lines.length < 2) throw new Error("데이터가 비어있거나 형식이 잘못되었습니다.");
        
        // BOM 제거 및 헤더 정리
        const headers = lines[0].replace(/^\uFEFF/, '').split(',').map(h => h.trim());
        if (!headers.includes('Lon') || !headers.includes('Lat')) throw new Error("필수 컬럼 누락: 'Lon', 'Lat'이 필요합니다.");

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
        return isMatch 
            ? { text: '분해점검', class: 'bg-rose-50 text-rose-700 border-rose-200' }
            : { text: '필터점검', class: 'bg-emerald-50 text-emerald-700 border-emerald-200' };
    }

    function handleFileSelect(event) {
        const file = event.target.files[0];
        const status = document.getElementById('fileStatus');
        const errDiv = document.getElementById('errorMsg');
        
        errDiv.classList.add('hidden');
        if (!file) return;

        status.textContent = `로드 중...`;
        const reader = new FileReader();
        
        reader.onload = (e) => {
            try {
                initDashboard(e.target.result);
                status.textContent = file.name;
                status.classList.add('text-teal-600', 'font-bold');
            } catch (error) {
                console.error(error);
                errDiv.textContent = `오류: ${error.message}`;
                errDiv.classList.remove('hidden');
                status.textContent = '실패';
                setTimeout(() => errDiv.classList.add('hidden'), 3000);
            }
        };
        reader.readAsText(file, 'euc-kr');
    }

    // --- UI Logic ---
    function createRegulatorCard(r, loopColor) {
        const card = document.createElement('div');
        card.className = 'bg-white rounded-xl shadow-sm border border-slate-200 p-4 hover:border-teal-500 hover:shadow-md transition-all cursor-pointer group';
        
        const isPriority = r.isPriority;
        const titleClass = isPriority ? 'text-rose-600' : 'text-slate-900';
        
        card.innerHTML = `
            <div class="flex justify-between items-start mb-3">
                <div class="flex items-center gap-3 overflow-hidden">
                    <span class="flex-shrink-0 flex items-center justify-center w-8 h-8 rounded-full text-white text-sm font-bold shadow-sm" style="background-color: ${loopColor};">
                        ${r.LOOP === 'default' ? '-' : r.LOOP}
                    </span>
                    <div class="min-w-0">
                        <h4 class="text-base font-bold ${titleClass} truncate leading-tight">${r['정압기명']}</h4>
                        <p class="text-xs text-slate-500 mt-0.5 truncate">${r['모델명'] || '모델 미정'}</p>
                    </div>
                </div>
                <div class="flex flex-col items-end gap-1 flex-shrink-0">
                    ${isPriority ? '<span class="px-1.5 py-0.5 bg-rose-50 text-rose-700 text-[11px] font-bold rounded border border-rose-100">우선방출</span>' : ''}
                    <span class="px-2 py-0.5 text-[11px] font-bold rounded border ${r.inspection.class}">${r.inspection.text}</span>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-2 mb-2 bg-slate-50 p-2.5 rounded-lg border border-slate-100">
                <div>
                    <span class="text-[11px] text-slate-500 flex items-center mb-0.5">${icons.pressure} 입/출구 압력</span>
                    <div class="font-mono text-sm font-bold text-slate-800 truncate tracking-tight">
                        ${r['입구압력(Mpa)']} <span class="text-slate-400 mx-0.5">/</span> ${r['출구압력(kPa)']}
                    </div>
                </div>
                <div class="border-l border-slate-200 pl-2">
                    <span class="text-[11px] text-slate-500 flex items-center mb-0.5">${icons.valve} 개도율</span>
                    <div class="font-mono text-sm font-bold text-teal-600">
                        ${r['센싱밸브 개도율'] || '-'}%
                    </div>
                </div>
            </div>

            <div class="flex justify-between items-center text-xs text-slate-500 pt-2 border-t border-slate-100">
                <div class="flex items-center gap-2 truncate">
                    <span class="flex items-center text-slate-600 font-medium">${icons.user} ${r['담당자']}</span>
                    <span class="text-slate-300">|</span>
                    <span class="truncate">${r['구간명']}</span>
                </div>
                <span class="text-slate-400 flex-shrink-0 font-mono tracking-tighter">${r['시공감리일자']}</span>
            </div>
        `;
        
        card.addEventListener('click', () => {
            if (map) {
                map.setView([r.Lat, r.Lon], 17, { animate: true });
                // 모바일에서 클릭 시 지도가 보이도록 상단 스크롤
                if (window.innerWidth <= 1024) {
                    document.querySelector('.app-container').scrollTo({ top: 0, behavior: 'smooth' });
                }
            }
        });
        return card;
    }

    function renderData(data) {
        const container = document.getElementById('regulator-cards');
        const countEl = document.getElementById('list-count');
        
        container.innerHTML = '';
        markers.clearLayers();
        countEl.textContent = `${data.length}건`;

        if (data.length === 0) {
            container.innerHTML = `<div class="text-center py-10 text-slate-400 text-sm">조건에 맞는 데이터가 없습니다.</div>`;
            return;
        }

        const bounds = [];
        data.forEach(r => {
            const loop = r.LOOP || 'default';
            const color = LOOP_COLORS[loop] || LOOP_COLORS['default'];
            
            container.appendChild(createRegulatorCard(r, color));

            if (!isNaN(r.Lat) && !isNaN(r.Lon)) {
                const iconHtml = `<div class="regulator-icon" style="background-color: ${color}; width: 36px; height: 36px;">${loop}</div>`;
                const icon = L.divIcon({
                    className: '',
                    html: iconHtml,
                    iconSize: [36, 36],
                    iconAnchor: [18, 18],
                    popupAnchor: [0, -10]
                });
                
                const popupContent = `
                    <div class="p-1 min-w-[180px]">
                        <div class="font-bold text-sm mb-1 text-slate-800">${r['정압기명']}</div>
                        <div class="text-xs text-slate-500 mb-2">${r['구간명']}</div>
                        <div class="text-xs border-t border-slate-200 pt-2 space-y-1">
                            <div class="flex justify-between"><span>입구:</span> <b class="font-mono">${r['입구압력(Mpa)']}</b></div>
                            <div class="flex justify-between"><span>출구:</span> <b class="font-mono">${r['출구압력(kPa)']}</b></div>
                            <div class="flex justify-between text-teal-600"><span>개도율:</span> <b class="font-mono">${r['센싱밸브 개도율']}%</b></div>
                        </div>
                    </div>
                `;

                const m = L.marker([r.Lat, r.Lon], { icon: icon }).bindPopup(popupContent);
                markers.addLayer(m);
                bounds.push([r.Lat, r.Lon]);
            }
        });

        if (bounds.length && map) {
            map.fitBounds(bounds, { padding: [50, 50], maxZoom: 16 });
        }
    }

    function populateFilters() {
        const els = ['loopFilter', 'modelFilter', 'sectionFilter'].map(id => document.getElementById(id));
        els.forEach(el => el.disabled = false);
        els.forEach(sel => { while(sel.options.length > 1) sel.remove(1); });

        const loops = [...new Set(regulators.map(r => r.LOOP))].filter(l => l !== 'default').sort((a,b) => parseInt(a)-parseInt(b));
        const models = [...new Set(regulators.map(r => r['정압기모델']).filter(Boolean))].sort();
        const sections = [...new Set(regulators.map(r => r['구간명']).filter(Boolean))].sort();

        const [elLoop, elModel, elSection] = els;
        loops.forEach(v => elLoop.add(new Option(`LOOP ${v}`, v)));
        sections.forEach(v => elSection.add(new Option(v, v)));
        models.forEach(v => elModel.add(new Option(v, v)));
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
        if (map) return;
        map = L.map('map', { zoomControl: false }).setView([36.77, 126.45], 11);
        L.control.zoom({ position: 'topright' }).addTo(map);
        L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
            attribution: '&copy; OpenStreetMap',
            subdomains: 'abcd',
            maxZoom: 20
        }).addTo(map);
        markers.addTo(map);
        
        // 레이아웃 변경 시 지도 깨짐 방지
        setTimeout(() => map.invalidateSize(), 300);
        window.addEventListener('resize', () => map.invalidateSize());
    }

    document.addEventListener('DOMContentLoaded', () => {
         initMap();
    });
</script>
</body>
</html>
