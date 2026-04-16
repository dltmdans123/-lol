<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LoL Real-time Strategist</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #020617;
        }
        .animate-pulse-slow {
            animation: pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: .5; }
        }
    </style>
</head>
<body class="text-slate-200">
    <div id="app" class="min-h-screen p-4 md:p-8">
        <!-- Header -->
        <header class="max-w-6xl mx-auto flex flex-col md:flex-row justify-between items-start md:items-center mb-8 border-b border-slate-800 pb-6 gap-4">
            <div>
                <h1 class="text-3xl font-bold text-blue-400 flex items-center gap-3">
                    <i data-lucide="zap" class="fill-blue-400 text-blue-400"></i>
                    LoL Real-time Strategist
                </h1>
                <p class="text-slate-400 mt-1">라이엇 API 기반 실시간 아이템 추천 엔진 (초안)</p>
            </div>
            <div class="flex gap-3">
                <div id="sync-status" class="flex items-center gap-2 px-4 py-2 bg-slate-800 rounded-lg text-sm font-medium border border-slate-700">
                    <div class="w-2 h-2 rounded-full bg-slate-500" id="status-dot"></div>
                    <span id="status-text">오프라인</span>
                </div>
                <button onclick="toggleSync()" id="sync-btn" class="flex items-center gap-2 px-5 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-lg font-bold transition-all shadow-lg shadow-blue-900/20">
                    <i data-lucide="refresh-cw" size="18" id="sync-icon"></i>
                    데이터 동기화 시작
                </button>
            </div>
        </header>

        <main class="max-w-6xl mx-auto grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- Left Panel: Analysis -->
            <div class="space-y-6">
                <section class="bg-slate-900 border border-slate-800 rounded-2xl p-6">
                    <h2 class="text-lg font-bold mb-5 flex items-center gap-2 text-red-400">
                        <i data-lucide="crosshair" size="20"></i> 적군 위협 요소
                    </h2>
                    <div class="space-y-4" id="threat-list">
                        <!-- Threat items will be injected here -->
                    </div>
                </section>

                <section class="bg-slate-900 border border-slate-800 rounded-2xl p-6">
                    <h2 class="text-lg font-bold mb-4 flex items-center gap-2 text-slate-300">
                        <i data-lucide="user" size="20"></i> 플레이어 상태
                    </h2>
                    <div class="bg-slate-800/50 p-4 rounded-xl border border-slate-700 flex items-center gap-4">
                        <div class="w-14 h-14 bg-gradient-to-br from-blue-600 to-blue-800 rounded-full flex items-center justify-center text-2xl font-black border-2 border-blue-400 shadow-inner">
                            E
                        </div>
                        <div>
                            <div class="font-bold text-lg">이즈리얼 (Ezreal)</div>
                            <div class="text-yellow-400 font-mono font-bold">1,450 G 보유 중</div>
                        </div>
                    </div>
                </section>
            </div>

            <!-- Right Panel: Recommendations -->
            <div class="lg:col-span-2 space-y-6">
                <section class="bg-slate-900 border border-blue-900/30 rounded-2xl p-6 shadow-xl">
                    <div class="flex justify-between items-center mb-6">
                        <h2 class="text-xl font-bold flex items-center gap-2">
                            <i data-lucide="sword" class="text-blue-400"></i> 상황별 최적 아이템
                        </h2>
                        <span class="text-xs text-slate-500 uppercase tracking-widest bg-slate-800 px-3 py-1 rounded-full">Top Recommendations</span>
                    </div>
                    
                    <div id="recommendation-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <!-- Item cards will be injected here -->
                    </div>
                </section>

                <div class="bg-blue-900/10 border border-blue-800/30 p-5 rounded-2xl flex gap-4">
                    <i data-lucide="alert-triangle" class="text-blue-500 shrink-0"></i>
                    <div>
                        <h4 class="text-sm font-bold text-blue-400 mb-1">개발자 참고 사항</h4>
                        <p class="text-xs text-slate-400 leading-relaxed">
                            이 화면은 UI/UX 프로토타입입니다. 실제 라이엇 API와 연동하려면 로컬 서버(Node.js 등)를 통해 <code>https://127.0.0.1:2999</code>의 인증서 보안을 우회하여 데이터를 가져와야 합니다. 
                        </p>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <script>
        // --- Data & Logic ---
        const MOCK_DATA = {
            threats: [
                { id: 'heal', label: '치유/회복', active: true, color: 'text-green-400', desc: '소라카/문도 포함됨' },
                { id: 'tank', label: '탱커 비중', active: true, color: 'text-yellow-400', desc: '적군 2인 이상 탱템 보유' },
                { id: 'burst', label: '암살자/버스트', active: true, color: 'text-red-400', desc: '제드(Zed) 성장세 높음' },
                { id: 'cc', label: '군중 제어(CC)', active: false, color: 'text-purple-400', desc: '위협적이지 않음' }
            ],
            items: [
                { 
                    name: '필멸자의 운명', 
                    tags: ['치유 감소', '방어구 관통'], 
                    reason: '상대 팀에 강력한 회복 수단(소라카)이 감지되었습니다. 치유 감소 효과가 필수적입니다.',
                    priority: 'High'
                },
                { 
                    name: '도미닉 경의 인사', 
                    tags: ['탱커 대응', '방어구 관통'], 
                    reason: '상대 팀의 사이온과 람머스가 높은 추가 체력을 보유하고 있습니다.',
                    priority: 'Essential'
                },
                { 
                    name: '몰락한 왕의 검', 
                    tags: ['체력 비례 데미지', '유틸리티'], 
                    reason: '상대 탱커들의 체력 바가 3000을 초과하여 현재 체력 비례 데미지가 효율적입니다.',
                    priority: 'Recommended'
                }
            ]
        };

        let isSyncing = false;

        function renderThreats() {
            const container = document.getElementById('threat-list');
            container.innerHTML = MOCK_DATA.threats.map(t => `
                <div class="flex justify-between items-center p-3 rounded-xl border border-slate-800 ${t.active ? 'bg-slate-800/40' : 'opacity-40'}">
                    <div>
                        <div class="text-sm font-bold ${t.active ? 'text-slate-200' : 'text-slate-500'}">${t.label}</div>
                        <div class="text-[10px] text-slate-500">${t.desc}</div>
                    </div>
                    <span class="text-[10px] font-black px-2 py-1 rounded border ${t.active ? `${t.color} border-current bg-current/5` : 'text-slate-600 border-slate-700'}">
                        ${t.active ? 'DANGER' : 'SAFE'}
                    </span>
                </div>
            `).join('');
        }

        function renderItems() {
            const container = document.getElementById('recommendation-grid');
            container.innerHTML = MOCK_DATA.items.map(item => `
                <div class="group bg-slate-800/40 hover:bg-slate-800 p-5 rounded-xl border border-slate-700 hover:border-blue-500/50 transition-all cursor-default">
                    <div class="flex justify-between items-start mb-3">
                        <h3 class="font-bold text-blue-100 group-hover:text-blue-400 transition-colors">${item.name}</h3>
                        <span class="text-[9px] font-bold bg-blue-500/10 text-blue-400 px-2 py-0.5 rounded border border-blue-500/20">${item.priority}</span>
                    </div>
                    <div class="flex gap-2 mb-4">
                        ${item.tags.map(tag => `<span class="text-[10px] text-slate-400 bg-slate-700 px-2 py-0.5 rounded">#${tag}</span>`).join('')}
                    </div>
                    <div class="flex gap-3 text-sm text-slate-400 leading-relaxed">
                        <i data-lucide="info" size="16" class="shrink-0 text-blue-500"></i>
                        ${item.reason}
                    </div>
                </div>
            `).join('');
            lucide.createIcons();
        }

        function toggleSync() {
            isSyncing = !isSyncing;
            const btn = document.getElementById('sync-btn');
            const icon = document.getElementById('sync-icon');
            const dot = document.getElementById('status-dot');
            const text = document.getElementById('status-text');

            if (isSyncing) {
                btn.classList.replace('bg-blue-600', 'bg-green-600');
                btn.innerHTML = `<i data-lucide="refresh-cw" size="18" class="animate-spin"></i> 실시간 분석 중...`;
                dot.classList.replace('bg-slate-500', 'bg-green-500');
                dot.classList.add('animate-pulse');
                text.innerText = '동기화 활성화';
            } else {
                btn.classList.replace('bg-green-600', 'bg-blue-600');
                btn.innerHTML = `<i data-lucide="refresh-cw" size="18"></i> 데이터 동기화 시작`;
                dot.classList.replace('bg-green-500', 'bg-slate-500');
                dot.classList.remove('animate-pulse');
                text.innerText = '오프라인';
            }
            lucide.createIcons();
        }

        // Initialize
        window.onload = () => {
            renderThreats();
            renderItems();
            lucide.createIcons();
        };
    </script>
</body>
</html>
