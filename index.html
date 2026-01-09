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
        /* 1. 강제 초기화: 여백 제거 및 전체 너비 사용 */
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            font-family: 'Pretendard', sans-serif;
            background-color: #f1f5f9;
        }

        /* 2. PC/모바일 분기 처리 */
        /* PC에서는 지도가 스크롤 따라오게(Sticky) 설정 */
        @media (min-width: 1024px) {
            #map-section {
                position: sticky;
                top: 64px; /* 헤더 높이만큼 띄움 */
                height: calc(100vh - 64px); /* 화면 전체 높이 사용 */
            }
        }
        
        /* 모바일에서는 지도가 그냥 위에 얹혀있음 */
        @media (max-width: 1023px) {
            #map-section {
                position: relative;
                height: 50vh; /* 화면 절반 */
                width: 100%;
            }
        }

        /* 아이콘 스타일 */
        .regulator-icon {
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 800;
            font-size: 14px;
            color: white;
            border-radius: 50%;
            border: 2px solid white;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
            transition: transform 0.2s;
        }
        .regulator-icon:hover { transform: scale(1.2); z-index: 1000 !important; }

        /* Select 스타일 */
        select {
            -webkit-appearance: none; appearance: none;
            background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3e%3cpath stroke='%2364748b' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3e%3c/svg%3e");
            background-position: right 0.75rem center; background-repeat: no-repeat; background-size: 1.25em 1.25em;
            padding-right: 2.5rem;
        }
    </style>
</head>

<body class="flex flex-col min-h-screen">

    <header class="sticky top-0 z-[999] w-full h-16 bg-slate-900 text-white flex items-center justify-between px-5 shadow-lg flex-none">
        <div class="flex items-center gap-3">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 text-teal-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                <path stroke-linecap="round" stroke-linejoin="round" d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0121 18.382V7.618a1 1 0 01-1.447-.894L15 7m0 13V7m0 0L9 4" />
            </svg>
            <h1 class="text-lg font-bold whitespace-nowrap">서부안전팀 대시보드</h1>
        </div>
        
        <div class="flex items-center gap-3">
            <span id="fileStatus" class="hidden md:block text-sm text-slate-400">파일 없음</span>
            <label for="csvFileInput" class="cursor-pointer bg-teal-600 hover:bg-teal-500 text-white px-4 py-2 rounded text-sm font-bold shadow flex items-center gap-2 whitespace-nowrap">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" /></svg>
                CSV 업로드
            </label>
            <input type="file" id="csvFileInput" accept=".csv" class="hidden" onchange="handleFileSelect(event)">
        </div>
    </header>

    <main class="w-full flex flex-col lg:flex-row flex-1">
        
        <section id="map-section" class="w-full lg:w-[65%] bg-slate-200 z-0">
            <div id="map" class="w-full h-full"></div>
            
            <div class="absolute top-4 left-4 z-[400] bg-white/95 backdrop-blur px-3 py-2 rounded shadow border border-slate-300">
                <h2 class="text-sm font-bold text-slate-800 flex items-center gap-2">
                    <span class="w-2.5 h-2.5 rounded-full bg-teal-500 animate-pulse"></span>
                    GIS 모니터링
                </h2>
            </div>
            
            <div id="errorMsg" class="absolute top-4 left-1/2 transform -translate-x-1/2 z-[1000] bg-rose-600 text-white px-4 py-2 rounded shadow-lg font-bold text-sm hidden"></div>
        </section>

        <section class="w-full lg:w-[35%] bg-slate-50 border-l border-slate-300 flex flex-col min-h-full z-10">
            
            <div class="p-5 bg-white border-b border-slate-200 shadow-sm">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-bold text-slate-800 flex items-center gap-2">
                        <svg class="w-5 h-5 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z"></path></svg>
                        필터
                    </h3>
                    <button onclick="resetFilters()" class="text-sm text-slate-500 underline hover:text-teal-600">초기화</button>
                </div>
                <div class="space-y-2">
                    <div class="flex gap-2">
                        <select id="loopFilter" onchange="applyFilters()" class="flex-1 bg-white border border-slate-300 text-slate-900 text-sm rounded p-2.5 disabled:bg-slate-100" disabled><option value="all">LOOP</option></select>
                        <select id="sectionFilter" onchange="applyFilters()" class="flex-1 bg-white border border-slate-300 text-slate-900 text-sm rounded p-2.5 disabled:bg-slate-100" disabled><option value="all">구간명</option></select>
                    </div>
                    <select id="modelFilter" onchange="applyFilters()" class="w-full bg-white border border-slate-300 text-slate-900 text-sm rounded p-2.5 disabled:bg-slate-100" disabled><option value="all">모델 전체</option></select>
                </div>
            </div>

            <div class="px-5 py-2 bg-slate-100 border-b border-slate-200 flex justify-between items-center">
                <span class="text-sm font-bold text-slate-600">목록</span>
                <span id="list-count" class="bg-teal-600 text-white text-xs font-bold px-2 py-1 rounded">0개</span>
            </div>

            <div id="regulator-cards" class="p-4 flex flex-col gap-3 flex-1">
                <div class="py-20 text-center text-slate-400">
                    <p class="text-base">CSV 파일을 업로드해주세요.</p>
                </div>
            </div>
        </section>

    </main>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha2d-p4z3r8gq333+7c3s2c4v4q0d1d2s3d4t5u6w7x8y9z0a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z" crossorigin=""></script>
    <script>
        // --- 설정 ---
        let regulators = [];
        let map = null;
        let markers = L.layerGroup(); 
        const LOOP_COLORS = {'1':'#3b82f6','2':'#10b981','3':'#f59e0b','4':'#8b5cf6','5':'#ef4444','6':'#06b6d4','7':'#ec4899','8':'#f97316','9':'#6366f1','10':'#14b8a6','default':'#64748b'};
        const icons = {
            pressure: `<svg class="w-3.5 h-3.5 text-slate-400 mr-1.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"/></svg>`,
            valve: `<svg class="w-3.5 h-3.5 text-slate-400 mr-1.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6V4m0 2a2 2 0 100 4m0-4a2 2 0 110 4m-6 8a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4m6 6v10m6-2a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4"/></svg>`,
            user: `<svg class="w-3.5 h-3.5 text-slate-400 mr-1.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"/></svg>`
        };

        // --- 로직 ---
        function parseCSV(csv){
            const lines = csv.trim().split('\n').filter(l=>l.trim()!=='');
            if(lines.length<2) throw new Error("데이터가 부족합니다.");
            const h = lines[0].replace(/^\uFEFF/,'').split(',').map(x=>x.trim());
            if(!h.includes('Lon')||!h.includes('Lat')) throw new Error("'Lon','Lat' 컬럼 필수");
            return lines.slice(1).map(line=>{
                const v = line.split(',');
                let o={}; h.forEach((k,i)=>o[k]=v[i]?v[i].trim():''); return o;
            });
        }

        function getInsp(d){
            const y = parseInt(d.substring(0,4))||new Date().getFullYear();
            return (new Date().getFullYear()%2 === y%2) 
                ? {t:'분해점검', c:'bg-rose-50 text-rose-700 border-rose-200'}
                : {t:'필터점검', c:'bg-emerald-50 text-emerald-700 border-emerald-200'};
        }

        function handleFileSelect(e){
            const f = e.target.files[0];
            const st = document.getElementById('fileStatus');
            const er = document.getElementById('errorMsg');
            er.classList.add('hidden');
            if(!f) return;
            st.textContent = "로드 중...";
            const r = new FileReader();
            r.onload = (ev) => {
                try {
                    initDash(ev.target.result);
                    st.textContent = f.name;
                    st.className = "text-teal-600 font-bold hidden md:block";
                } catch(err) {
                    er.textContent = err.message; er.classList.remove('hidden');
                    st.textContent = "실패";
                    setTimeout(()=>er.classList.add('hidden'), 3000);
                }
            };
            r.readAsText(f, 'euc-kr');
        }

        function createCard(r, c){
            const d = document.createElement('div');
            d.className = 'bg-white rounded-lg shadow-sm border border-slate-200 p-4 hover:border-teal-500 hover:shadow-md cursor-pointer transition-all';
            const isP = r.isPriority;
            d.innerHTML = `
                <div class="flex justify-between items-start mb-3">
                    <div class="flex items-center gap-3">
                        <span class="flex-shrink-0 w-8 h-8 rounded-full flex items-center justify-center text-white text-xs font-bold shadow-sm" style="background-color:${c}">${r.LOOP==='default'?'-':r.LOOP}</span>
                        <div>
                            <h4 class="text-sm font-bold ${isP?'text-rose-600':'text-slate-900'}">${r['정압기명']}</h4>
                            <p class="text-xs text-slate-500">${r['모델명']||'-'}</p>
                        </div>
                    </div>
                    <div class="flex flex-col items-end gap-1">
                        ${isP?`<span class="px-1.5 py-0.5 bg-rose-50 text-rose-700 text-[11px] font-bold rounded border border-rose-100">우선방출</span>`:''}
                        <span class="px-1.5 py-0.5 text-[11px] font-bold rounded border ${r.inspection.c}">${r.inspection.t}</span>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-2 mb-2 bg-slate-50 p-2 rounded border border-slate-100">
                    <div><span class="text-[11px] text-slate-500 flex items-center">${icons.pressure} 압력</span><div class="font-mono text-xs font-bold text-slate-700 mt-0.5">${r['입구압력(Mpa)']} / ${r['출구압력(kPa)']}</div></div>
                    <div class="border-l pl-2 border-slate-200"><span class="text-[11px] text-slate-500 flex items-center">${icons.valve} 개도율</span><div class="font-mono text-xs font-bold text-teal-600 mt-0.5">${r['센싱밸브 개도율']||'-'}%</div></div>
                </div>
                <div class="flex justify-between text-xs text-slate-500 pt-1 border-t border-slate-100">
                    <div class="flex gap-2 items-center"><span>${icons.user} ${r['담당자']}</span><span class="text-slate-300">|</span><span>${r['구간명']}</span></div>
                    <span>${r['시공감리일자']}</span>
                </div>
            `;
            d.onclick = () => {
                if(map) {
                    map.setView([r.Lat, r.Lon], 17, {animate:true});
                    window.scrollTo({top:0, behavior:'smooth'});
                }
            };
            return d;
        }

        function render(data){
            const c = document.getElementById('regulator-cards');
            const cnt = document.getElementById('list-count');
            c.innerHTML = ''; markers.clearLayers();
            cnt.textContent = `${data.length}개`;
            
            if(data.length===0) { c.innerHTML = `<div class="text-center py-10 text-slate-400">결과 없음</div>`; return; }
            
            const b = [];
            data.forEach(r => {
                const lp = r.LOOP||'default';
                const col = LOOP_COLORS[lp]||LOOP_COLORS['default'];
                c.appendChild(createCard(r, col));
                
                if(!isNaN(r.Lat) && !isNaN(r.Lon)){
                    const ic = L.divIcon({className:'', html:`<div class="regulator-icon" style="background-color:${col}; width:32px; height:32px;">${lp}</div>`, iconSize:[32,32], iconAnchor:[16,16]});
                    const pop = `<div class="p-1 min-w-[150px]"><div class="font-bold mb-1">${r['정압기명']}</div><div class="text-xs text-slate-500">${r['구간명']}</div></div>`;
                    markers.addLayer(L.marker([r.Lat, r.Lon], {icon:ic}).bindPopup(pop));
                    b.push([r.Lat, r.Lon]);
                }
            });
            if(b.length && map) map.fitBounds(b, {padding:[50,50], maxZoom:16});
        }

        function applyFilters(){
            const vl = document.getElementById('loopFilter').value;
            const vs = document.getElementById('sectionFilter').value;
            const vm = document.getElementById('modelFilter').value;
            const res = regulators.filter(r => {
                return (vl==='all'||r.LOOP==vl) && (vs==='all'||r['구간명']===vs) && (vm==='all'||r['정압기모델']===vm);
            });
            render(res);
        }
        function resetFilters(){
            ['loopFilter','sectionFilter','modelFilter'].forEach(id=>document.getElementById(id).value='all');
            applyFilters();
        }

        function initDash(csv){
            const raw = parseCSV(csv);
            regulators = raw.map(r => {
                let l = r.LOOP?r.LOOP.trim():'default';
                if(!isNaN(parseInt(l))) l = parseInt(l).toString();
                return { ...r, Lon:parseFloat(r.Lon), Lat:parseFloat(r.Lat), isPriority:r['비고']&&r['비고'].includes('우선'), LOOP:l, inspection:getInsp(r['시공감리일자']) };
            });
            
            const els = ['loopFilter','sectionFilter','modelFilter'].map(id=>document.getElementById(id));
            els.forEach(e=>{ e.disabled=false; while(e.options.length>1)e.remove(1); });
            
            const lps = [...new Set(regulators.map(r=>r.LOOP))].filter(l=>l!=='default').sort((a,b)=>parseInt(a)-parseInt(b));
            const secs = [...new Set(regulators.map(r=>r['구간명']).filter(Boolean))].sort();
            const mods = [...new Set(regulators.map(r=>r['정압기모델']).filter(Boolean))].sort();
            
            lps.forEach(v=>els[0].add(new Option(`LOOP ${v}`,v)));
            secs.forEach(v=>els[1].add(new Option(v,v)));
            mods.forEach(v=>els[2].add(new Option(v,v)));
            
            if(!map) initMap();
            render(regulators);
        }

        function initMap(){
            if(map) return;
            map = L.map('map',{zoomControl:false}).setView([36.77, 126.45], 11);
            L.control.zoom({position:'topright'}).addTo(map);
            L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {attribution:'&copy; OpenStreetMap', maxZoom:20}).addTo(map);
            markers.addTo(map);
            setTimeout(()=>map.invalidateSize(), 500);
        }

        document.addEventListener('DOMContentLoaded', ()=>initMap());
    </script>
</body>
</html>
