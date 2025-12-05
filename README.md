<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Supermarket Design Challenge</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Chosen Palette: Warm Neutrals & Fresh Accents -->
    <!-- Palette Description: Backgrounds in Cream (#FDFBF7) and Warm Grey (#F5F5F4). Accents in Sage Green (#84A98C) and Terra Cotta (#E76F51) for interactive elements. Text in Dark Charcoal (#2D3748) for readability. -->
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

    <!-- Application Structure Plan: 
         1. Header & Setup: Users name their supermarket. Sets the context.
         2. Design Studio (Interactive Map): The core "Part 2" activity. A drag-and-drop style interaction (click-to-place) where users assign the 11 inventory items to grid slots. This allows for spatial planning.
         3. Logic Lab (Part 3): Interactive quiz form for the multiple-choice reasoning questions. Validates understanding of store layout psychology.
         4. Strategy Room (Creative): Text area for the open-ended strategy question.
         5. Dashboard (Review): A summary view. It visualizes the "Score" (simulated based on completion and logic accuracy) using Chart.js to provide instant feedback and a "Gamified" feel. 
         Rationale: This linear but interactive flow mimics the worksheet's structure but enhances it with instant visual feedback (filling the map) and data visualization (scoring), making the abstract concepts concrete.
    -->

    <!-- Visualization & Content Choices:
         - Layout Map: HTML Grid. Goal: Organize. Visualizes spatial relationships.
         - Inventory List: Interactive List. Goal: Inform/Select. 
         - Logic Quiz: Form Elements. Goal: Analyze.
         - Score Chart: Chart.js Bar Chart. Goal: Inform/Result. Visualizes the breakdown of points (Naming, Layout, Logic, Creativity). Confirms NO SVG/Mermaid.
    -->
    
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->

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
            
            <div class="p-8 grid grid-cols-1 md:grid-cols-2 gap-8 items-center">
                <!-- Score Breakdown Chart -->
                <div class="flex flex-col items-center">
                    <h3 class="text-lg font-bold text-stone-700 mb-4">Performance Analysis</h3>
                    <div class="chart-container">
                        <canvas id="scoreChart"></canvas>
                    </div>
                </div>

                <!-- Text Summary -->
                <div class="space-y-6">
                    <div class="bg-stone-50 p-6 rounded-xl border border-stone-100">
                        <h4 class="text-sm font-bold text-stone-500 uppercase tracking-wide mb-2">Total Score</h4>
                        <div class="flex items-baseline space-x-2">
                            <span id="total-score" class="text-5xl font-extrabold text-stone-800">0</span>
                            <span class="text-stone-500">/ 50 Points</span>
                        </div>
                        <p id="score-message" class="mt-2 text-stone-600 italic">Complete the challenge to see your rank!</p>
                    </div>

                    <div class="space-y-2">
                        <h4 class="text-sm font-bold text-stone-500 uppercase tracking-wide">Review Details</h4>
                        <ul class="text-sm text-stone-700 space-y-1">
                            <li class="flex justify-between"><span>Inventory Placement:</span> <span id="score-placement">0/15</span></li>
                            <li class="flex justify-between"><span>Logic Quiz:</span> <span id="score-logic">0/9</span></li>
                            <li class="flex justify-between"><span>Creativity (Est.):</span> <span id="score-creative">0/16</span></li>
                            <li class="flex justify-between"><span>Peer Review Bonus:</span> <span id="score-peer">10/10</span></li>
                        </ul>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <footer class="bg-stone-100 border-t border-stone-200 mt-auto py-8">
        <div class="max-w-6xl mx-auto px-4 text-center text-stone-500 text-sm">
            <p>&copy; 2025 ESL Interactive Learning. Based on "Supermarket Design Challenge".</p>
        </div>
    </footer>

    <script>
        // Data & State
        const inventoryItems = [
            { id: 'meat', name: 'Meat (고기)', icon: '🥩', cat: 'fresh' },
            { id: 'poultry', name: 'Poultry (닭고기)', icon: '🍗', cat: 'fresh' },
            { id: 'seafood', name: 'Seafood (해산물)', icon: '🦐', cat: 'fresh' },
            { id: 'frozen', name: 'Frozen Foods', icon: '❄️', cat: 'frozen' },
            { id: 'dairy', name: 'Dairy (유제품)', icon: '🥛', cat: 'fresh' },
            { id: 'beverages', name: 'Beverages (음료)', icon: '🧃', cat: 'dry' },
            { id: 'snack', name: 'Snacks (간식)', icon: '🍪', cat: 'dry' },
            { id: 'canned', name: 'Canned Foods', icon: '🥫', cat: 'dry' },
            { id: 'sauces', name: 'Sauces (소스)', icon: '🍯', cat: 'dry' },
            { id: 'grains', name: 'Grains (곡물)', icon: '🌾', cat: 'dry' },
            { id: 'dessert', name: 'Dessert (디저트)', icon: '🍰', cat: 'fresh' }
        ];

        let state = {
            marketName: "",
            placedItems: {}, // cellIndex -> itemObject
            selectedItem: null, // item currently clicked in sidebar
            answers: { q1: null, q2: null, q3: null },
            creativeText: ""
        };

        // DOM Elements
        const inventoryListEl = document.getElementById('inventory-list');
        const gridEl = document.getElementById('store-grid');
        const placedCountEl = document.getElementById('placed-count');
        const invCountEl = document.getElementById('inv-count');

        // Initialization
        function init() {
            renderInventory();
            renderGrid();
        }

        // Render Inventory List
        function renderInventory() {
            inventoryListEl.innerHTML = '';
            let remaining = 0;
            
            inventoryItems.forEach(item => {
                // Check if item is already placed
                const isPlaced = Object.values(state.placedItems).some(placed => placed.id === item.id);
                
                if (!isPlaced) {
                    remaining++;
                    const card = document.createElement('div');
                    card.className = `item-card flex items-center p-3 bg-white rounded-lg border border-stone-200 shadow-sm hover:shadow-md ${state.selectedItem && state.selectedItem.id === item.id ? 'selected' : ''}`;
                    card.innerHTML = `
                        <span class="text-2xl mr-3">${item.icon}</span>
                        <span class="text-sm font-medium text-stone-700">${item.name}</span>
                    `;
                    card.onclick = () => selectItem(item);
                    inventoryListEl.appendChild(card);
                }
            });

            invCountEl.textContent = remaining;
            placedCountEl.textContent = inventoryItems.length - remaining;
        }

        // Render Map Grid
        function renderGrid() {
            gridEl.innerHTML = '';
            // 3 cols x 4 rows = 12 cells (0 to 11)
            for (let i = 0; i < 12; i++) {
                const cell = document.createElement('div');
                const placedItem = state.placedItems[i];
                
                // For debugging grid indices, uncomment the line below:
                // const indexLabel = i.toString(); 

                cell.className = `grid-cell bg-white rounded-lg border-2 ${placedItem ? 'border-green-400 bg-green-50' : 'border-stone-200'} flex flex-col items-center justify-center p-2 min-h-[100px] cursor-pointer relative`;
                
                if (placedItem) {
                    cell.innerHTML = `
                        <span class="text-3xl mb-1">${placedItem.icon}</span>
                        <span class="text-xs font-bold text-center leading-tight text-stone-800">${placedItem.name.split('(')[0]}</span>
                        <button onclick="event.stopPropagation(); removeItem(${i})" class="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-5 h-5 flex items-center justify-center text-xs hover:bg-red-600 shadow-sm">×</button>
                    `;
                } else {
                    cell.innerHTML = `<span class="text-stone-300 text-2xl font-bold opacity-30">+</span>`;
                    cell.onclick = () => placeItemInCell(i);
                }
                
                gridEl.appendChild(cell);
            }
        }

        // Interaction Logic
        function selectItem(item) {
            state.selectedItem = item;
            renderInventory();
        }

        function placeItemInCell(index) {
            if (state.selectedItem) {
                // Check if the cell is empty
                if (state.placedItems[index]) {
                    // Remove existing item from cell first (optional but good UX)
                    delete state.placedItems[index];
                }

                // Check if the item is already placed elsewhere and remove it
                const existingIndex = Object.keys(state.placedItems).find(key => state.placedItems[key].id === state.selectedItem.id);
                if (existingIndex !== undefined) {
                    delete state.placedItems[existingIndex];
                }

                state.placedItems[index] = state.selectedItem;
                state.selectedItem = null;
                renderInventory();
                renderGrid();
            } else {
                // Translated alert message
                alert("왼쪽 인벤토리(Inventory)에서 배치할 품목을 먼저 선택해 주세요!");
            }
        }

        function removeItem(index) {
            delete state.placedItems[index];
            renderInventory();
            renderGrid();
        }

        function updateMarketName() {
            const input = document.getElementById('market-name-input');
            const display = document.getElementById('name-display');
            if(input.value.trim() !== "") {
                state.marketName = input.value;
                display.textContent = state.marketName;
                display.classList.remove('hidden');
                document.getElementById('result-market-name').textContent = state.marketName;
            } else {
                alert("Please enter a name!");
            }
        }

        // Scoring Logic
        function calculateResults() {
            // 1. Validation
            const placedCount = Object.keys(state.placedItems).length;
            if (placedCount < 1) {
                alert("슈퍼마켓 지도를 먼저 디자인해주세요! 최소한 몇 개의 품목이라도 배치해야 합니다.");
                return;
            }

            // --- 2. Logic Quiz Score (Correct answers based on source text context) ---
            // Q1: Meat need cold -> B (Near loading/cold storage)
            // Q2: Bev/Dairy similar -> B (Both need refrigeration)
            // Q3: Snacks impulse -> C (Quick/Spontaneous)
            let logicScore = 0;
            const q1Val = document.querySelector('input[name="q1"]:checked')?.value;
            const q2Val = document.querySelector('input[name="q2"]:checked')?.value;
            const q3Val = document.querySelector('input[name="q3"]:checked')?.value;

            if (q1Val === 'B') logicScore += 3;
            if (q2Val === 'B') logicScore += 3;
            if (q3Val === 'C') logicScore += 3;

            // --- 3. Placement Score (Logic-based, Max 15 points) ---
            let placementScore = 0;
            const placedItemsArray = Object.entries(state.placedItems); // [["0", {id: 'meat', ...}], ...]
            
            // Zones: Grid is 3x4 (0-11). Entrance/Checkout is bottom row (9, 10, 11). Loading/Back Wall is top row (0, 1, 2).
            const BACK_WALL_ZONE = [0, 1, 2];
            const IMPULSE_ZONE = [9, 10, 11]; // Near Checkout
            const PERIMETER_ZONE = [0, 1, 2, 3, 5, 6, 8, 9, 11]; // Walls for cold storage/flow

            // Key Item Categories
            const FRESH_COLD_ITEMS = ['meat', 'poultry', 'seafood', 'frozen', 'dairy', 'dessert'];
            const NECESSITY_ITEMS = ['grains', 'beverages'];
            const IMPULSE_ITEM_ID = 'snack';

            let backWallFreshCount = 0;
            
            placedItemsArray.forEach(([cellIndexStr, item]) => {
                const cellIndex = parseInt(cellIndexStr);
                
                // 3.1. Fresh/Cold Items (Max 9 pts)
                if (FRESH_COLD_ITEMS.includes(item.id)) {
                    // +1 point for being in the Perimeter Zone (Wall) - Principle 2: Fresh Goods Outer/Back Wall
                    if (PERIMETER_ZONE.includes(cellIndex)) {
                        placementScore += 1; // Max 6 pts (6 items * 1 pt)
                    }
                    
                    // Count for Back Wall Bonus (Principle 2)
                    if (BACK_WALL_ZONE.includes(cellIndex)) {
                        backWallFreshCount += 1;
                    }
                }
                
                // 3.2. Impulse Item (Max 2 pts)
                if (item.id === IMPULSE_ITEM_ID) {
                    // +2 points for being in the Impulse Zone (Front/Checkout) - Principle 3: Impulse Zone Strategy
                    if (IMPULSE_ZONE.includes(cellIndex)) {
                        placementScore += 2; // Max 2 pts
                    }
                }
            });

            // 3.1. Bonus for Fresh/Cold in Back Wall (Max 3 pts) - Efficiency & Cold Chain
            // 1 point for every 2 FRESH_COLD_ITEMS in the BACK_WALL_ZONE, up to 3 points max
            placementScore += Math.min(3, Math.floor(backWallFreshCount / 2));

            // 3.3. Necessity/Traffic Flow Items (Max 4 pts) - Principle 1: Traffic Flow Maximization
            // Check if both Grains AND Beverages are placed far from the Impulse Zone (9, 10, 11)
            const grainsPlacedFar = placedItemsArray.some(([cellIndexStr, item]) => 
                item.id === 'grains' && !IMPULSE_ZONE.includes(parseInt(cellIndexStr))
            );
            const beveragesPlacedFar = placedItemsArray.some(([cellIndexStr, item]) => 
                item.id === 'beverages' && !IMPULSE_ZONE.includes(parseInt(cellIndexStr))
            );
            
            if (grainsPlacedFar && beveragesPlacedFar) {
                // 2 points each for successful placement outside the impulse zone (far from entrance)
                placementScore += 4;
            } else if (grainsPlacedFar || beveragesPlacedFar) {
                // 2 points if only one is placed far
                placementScore += 2;
            }
            
            // Final Placement Score must be capped at 15
            placementScore = Math.min(15, placementScore);


            // 4. Creative Score (Check if text exists)
            const creativeText = document.getElementById('creative-input').value;
            // Simplified scoring: > 20 chars gets max, > 0 chars gets half, else 0.
            const creativeScore = creativeText.length > 20 ? 16 : (creativeText.length > 0 ? 8 : 0); 

            // 5. Peer Review (Fixed Bonus)
            const peerScore = 10;

            const totalScore = logicScore + placementScore + creativeScore + peerScore;

            // Update DOM
            document.getElementById('total-score').textContent = totalScore;
            document.getElementById('score-placement').textContent = `${placementScore}/15`;
            document.getElementById('score-logic').textContent = `${logicScore}/9`;
            document.getElementById('score-creative').textContent = `${creativeScore}/16`;
            document.getElementById('score-peer').textContent = `${peerScore}/10`;

            const messageEl = document.getElementById('score-message');
            if (totalScore >= 45) messageEl.textContent = "🏆 Outstanding! A Master Layout!";
            else if (totalScore >= 35) messageEl.textContent = "🥈 Great Job! Very logical design.";
            else messageEl.textContent = "🥉 Good effort! Keep improving your strategy.";

            // Show Results
            document.getElementById('results-dashboard').classList.remove('hidden');
            document.getElementById('results-dashboard').scrollIntoView({ behavior: 'smooth' });

            renderChart(placementScore, logicScore, creativeScore, peerScore);
        }

        // Chart.js Visualization
        let myChart = null;
        function renderChart(p, l, c, peer) {
            const ctx = document.getElementById('scoreChart').getContext('2d');
            
            if (myChart) myChart.destroy();

            myChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Layout (15)', 'Logic (9)', 'Creativity (16)', 'Peer Review (10)'],
                    datasets: [{
                        label: 'Your Score',
                        data: [p, l, c, peer],
                        backgroundColor: [
                            'rgba(132, 169, 140, 0.7)', // Sage
                            'rgba(231, 111, 81, 0.7)', // Terra Cotta
                            'rgba(233, 196, 106, 0.7)', // Yellow
                            'rgba(42, 157, 143, 0.7)'  // Teal
                        ],
                        borderColor: [
                            'rgba(132, 169, 140, 1)',
                            'rgba(231, 111, 81, 1)',
                            'rgba(233, 196, 106, 1)',
                            'rgba(42, 157, 143, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            max: 20
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        }
                    }
                }
            });
        }

        // Start
        init();

    </script>
</body>
</html>
