<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Supermarket Design Challenge</title>
    <!-- 1. Tailwind CSS (CDN - 스타일링에 필수) -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 2. Chart.js (CDN - 그래프 표시에 필수) -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #FDFBF7; color: #2D3748; }
        .chart-container { position: relative; width: 100%; max-width: 600px; margin-left: auto; margin-right: auto; height: 300px; max-height: 400px; }
        @media (min-width: 768px) { .chart-container { height: 350px; } }
        
        .grid-cell { transition: all 0.3s ease; }
        .grid-cell:hover { transform: scale(1.02); box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1); }
        .item-card { cursor: pointer; transition: all 0.2s; }
        .item-card.selected { ring: 2px; ring-color: #E76F51; background-color: #FFE4DE; transform: translateY(-2px); }
        .fade-in { animation: fadeIn 0.5s ease-in; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        
        /* Custom Scrollbar for inventory list */
        .custom-scroll::-webkit-scrollbar { width: 6px; }
        .custom-scroll::-webkit-scrollbar-track { background: #f1f1f1; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e0; border-radius: 3px; }
        .custom-scroll::-webkit-scrollbar-thumb:hover { background: #a0aec0; }
    </style>
</head>
<body class="min-h-screen flex flex-col">

    <!-- Header Section -->
    <header class="bg-white shadow-sm border-b border-stone-200 sticky top-0 z-50">
        <div class="max-w-6xl mx-auto px-4 py-4 flex justify-between items-center">
            <div class="flex items-center space-x-3">
                <span class="text-3xl">🛒</span>
                <div>
                    <h1 class="text-xl font-bold text-stone-800">Supermarket Design Challenge</h1>
                    <p class="text-sm text-stone-500">나만의 슈퍼마켓을 디자인하고 전략을 세워보세요!</p>
                </div>
            </div>
            <div id="progress-indicator" class="hidden md:flex items-center space-x-2 text-sm font-medium text-stone-600 bg-stone-100 px-3 py-1 rounded-full">
                <span>Items Placed: <span id="placed-count" class="text-green-600 font-bold">0</span>/11</span>
            </div>
        </div>
    </header>

    <main class="flex-grow max-w-6xl mx-auto px-4 py-8 space-y-12 w-full">

        <!-- Intro Section: Naming -->
        <section id="step-1" class="bg-white rounded-2xl shadow-sm border border-stone-100 p-6 md:p-8 fade-in">
            <div class="mb-6">
                <h2 class="text-2xl font-bold text-stone-800 mb-2">Step 1: Branding</h2>
                <p class="text-stone-600">여러분의 슈퍼마켓 이름을 정해주세요. 창의적인 이름은 고객을 끌어들입니다!</p>
            </div>
            <div class="flex flex-col md:flex-row gap-4 items-end">
                <div class="w-full md:w-1/2">
                    <label class="block text-sm font-semibold text-stone-700 mb-2">Supermarket Name</label>
                    <input type="text" id="market-name-input" placeholder="e.g., Fresh Mart, Happy Grocery" class="w-full p-3 border border-stone-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500 outline-none transition-all">
                </div>
                <div class="w-full md:w-auto">
                    <button onclick="updateMarketName()" class="w-full md:w-auto bg-stone-800 hover:bg-stone-700 text-white px-6 py-3 rounded-lg font-medium transition-colors">이름 확정 (Set Name)</button>
                </div>
            </div>
            <div id="name-display" class="mt-4 hidden text-lg text-green-700 font-bold bg-green-50 px-4 py-2 rounded-lg inline-block border border-green-100"></div>
        </section>

        <!-- Design Studio: Map & Inventory -->
        <section id="step-2" class="fade-in">
            <div class="mb-6">
                <h2 class="text-2xl font-bold text-stone-800 mb-2">Step 2: Store Layout Design</h2>
                <p class="text-stone-600">왼쪽의 품목(Inventory)을 선택하여 오른쪽 매장 지도(Store Map)의 원하는 위치에 배치하세요. <br> <span class="text-sm text-stone-500">* 전략적으로 생각하세요: 신선식품은 어디에? 간식은 어디에?</span></p>
            </div>

            <div class="flex flex-col lg:flex-row gap-6 h-auto lg:h-[600px]">
                <!-- Inventory Sidebar -->
                <div class="w-full lg:w-1/3 flex flex-col bg-white rounded-xl shadow-sm border border-stone-200 overflow-hidden">
                    <div class="p-4 bg-stone-50 border-b border-stone-200">
                        <h3 class="font-bold text-stone-700">📦 Inventory (<span id="inv-count">11</span> left)</h3>
                    </div>
                    <div id="inventory-list" class="flex-1 overflow-y-auto p-4 space-y-2 custom-scroll">
                        <!-- Items will be injected by JS -->
                    </div>
                </div>

                <!-- Map Grid -->
                <div class="w-full lg:w-2/3 bg-stone-100 rounded-xl border-2 border-dashed border-stone-300 p-6 flex flex-col relative">
                    <div class="absolute top-0 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-stone-800 text-white px-4 py-1 rounded-full text-xs font-bold uppercase tracking-wider shadow-sm">Main Entrance</div>
                    
                    <div class="flex-1 grid grid-cols-3 grid-rows-4 gap-4 mt-4" id="store-grid">
                        <!-- Grid cells will be generated by JS -->
                    </div>
                    
                    <div class="mt-4 flex justify-between text-xs text-stone-500 px-2">
                        <span>Checkout Area (Front)</span>
                        <span>Storage / Loading (Back)</span>
                    </div>
                </div>
            </div>
        </section>

        <!-- Logic Lab: Quiz -->
        <section id="step-3" class="bg-white rounded-2xl shadow-sm border border-stone-100 p-6 md:p-8 fade-in">
            <div class="mb-6">
                <h2 class="text-2xl font-bold text-stone-800 mb-2">Step 3: Logic & Reasoning</h2>
                <p class="text-stone-600">배치 전략에 대한 이유를 설명하세요. 가장 적절한 답변을 선택하세요.</p>
            </div>

            <div class="space-y-8">
                <!-- Question 1 -->
                <div class="bg-stone-50 p-6 rounded-xl border border-stone-200">
                    <h3 class="font-bold text-lg text-stone-800 mb-4">1. Meat, Poultry, and Seafood (육류/해산물)를 구석이나 안쪽에 놓은 이유는?</h3>
                    <div class="space-y-3">
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q1" value="A" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(A) They need to be <strong>far from</strong> the entrance because they smell bad.</span>
                        </label>
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q1" value="B" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(B) They need to be <strong>near</strong> the loading dock (or back wall) because they need cold storage/refrigeration.</span>
                        </label>
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q1" value="C" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(C) They are <strong>next to</strong> the Snack Foods for easy shopping.</span>
                        </label>
                    </div>
                </div>

                <!-- Question 2 -->
                <div class="bg-stone-50 p-6 rounded-xl border border-stone-200">
                    <h3 class="font-bold text-lg text-stone-800 mb-4">2. Beverages (음료)와 Dairy Products (유제품)를 가까이 혹은 벽면에 놓은 이유는?</h3>
                    <div class="space-y-3">
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q2" value="A" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(A) They are <strong>always</strong> placed near the Desserts.</span>
                        </label>
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q2" value="B" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(B) They are <strong>similar</strong> because they both often need refrigeration/coolers along the wall.</span>
                        </label>
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q2" value="C" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(C) Customers usually buy them <strong>after</strong> they buy snacks.</span>
                        </label>
                    </div>
                </div>

                <!-- Question 3 -->
                <div class="bg-stone-50 p-6 rounded-xl border border-stone-200">
                    <h3 class="font-bold text-lg text-stone-800 mb-4">3. Snack Foods (간식)를 계산대 근처나 통로 앞쪽에 놓는 이유는?</h3>
                    <div class="space-y-3">
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q3" value="A" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(A) It is <strong>far from</strong> the entrance for long shopping trips.</span>
                        </label>
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q3" value="B" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(B) Snacks are <strong>cheap</strong> so we want customers to find them easily.</span>
                        </label>
                        <label class="flex items-start space-x-3 p-3 bg-white rounded-lg border border-stone-200 cursor-pointer hover:border-blue-400 transition-colors">
                            <input type="radio" name="q3" value="C" class="mt-1 text-blue-600 focus:ring-blue-500">
                            <span class="text-sm text-stone-700">(C) We want customers to buy them <strong>quickly</strong> and spontaneously (Impulse buy).</span>
                        </label>
                    </div>
                </div>
            </div>
        </section>

        <!-- Creative Strategy -->
        <section id="step-4" class="bg-white rounded-2xl shadow-sm border border-stone-100 p-6 md:p-8 fade-in">
            <div class="mb-6">
                <h2 class="text-2xl font-bold text-stone-800 mb-2">Part 3B: Creative Idea</h2>
                <p class="text-stone-600">우리 팀만의 특별한 배치 아이디어를 영어로 적어주세요. (Write 2-3 sentences)</p>
            </div>
            <textarea id="creative-input" rows="4" class="w-full p-4 border border-stone-300 rounded-xl focus:ring-2 focus:ring-green-500 outline-none resize-none bg-stone-50" placeholder="Our special placement idea is... (e.g., We put the bakery next to the entrance so customers smell fresh bread first.)"></textarea>
        </section>

        <!-- Submit Action -->
        <div class="flex justify-center pb-8">
            <button onclick="calculateResults()" class="bg-gradient-to-r from-stone-800 to-stone-700 hover:from-stone-700 hover:to-stone-600 text-white text-xl font-bold px-12 py-4 rounded-full shadow-lg transform hover:scale-105 transition-all">
                ✨ Submit Design & Get Score
            </button>
        </div>

        <!-- Results Dashboard (Hidden initially) -->
        <section id="results-dashboard" class="hidden bg-white rounded-2xl shadow-xl border border-stone-200 overflow-hidden mb-12">
            <div class="bg-stone-800 p-6 text-white text-center">
                <h2 class="text-3xl font-bold mb-2">🏆 Design Report Card</h2>
                <p class="opacity-80">Supermarket: <span id="result-market-name" class="font-bold text-yellow-400">Unnamed</span></p>
            </div>
            
            <div class="p-8
