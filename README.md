[index.html](https://github.com/user-attachments/files/35391096/index.html)

<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>智慧記帳管家 - 行動記帳與預算控制</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Google Fonts - Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }
        /* Hide scrollbars for horizontal category pills on mobile */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        @keyframes pulse-border {
            0%, 100% { border-color: rgba(244, 63, 94, 0.4); }
            50% { border-color: rgba(244, 63, 94, 1); }
        }
        .overspend-pulse {
            animation: pulse-border 2s infinite;
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col pb-24 sm:pb-8">

    <!-- Notification Toast -->
    <div id="toast" class="fixed top-4 left-1/2 -translate-x-1/2 sm:left-auto sm:right-5 sm:translate-x-0 z-50 transform -translate-y-24 sm:translate-y-0 sm:translate-x-[150%] transition-transform duration-300 bg-slate-900 text-white px-5 py-3 rounded-2xl shadow-xl flex items-center space-x-2.5 max-w-[90vw]">
        <i id="toast-icon" class="fa-solid fa-circle-check text-emerald-400"></i>
        <span id="toast-message" class="text-sm font-medium">操作成功</span>
    </div>

    <!-- Top Sticky Header -->
    <header class="bg-white/90 backdrop-blur-md border-b border-slate-200/80 sticky top-0 z-30">
        <div class="max-w-4xl mx-auto px-4 py-3.5 flex flex-wrap justify-between items-center gap-3">
            <!-- Brand -->
            <div class="flex items-center space-x-2.5">
                <div class="bg-indigo-600 text-white w-10 h-10 rounded-xl shadow-sm flex items-center justify-center">
                    <i class="fa-solid fa-wallet text-lg"></i>
                </div>
                <div>
                    <h1 class="text-lg font-extrabold text-slate-900 tracking-tight leading-tight">智慧記帳管家</h1>
                    <p class="text-[11px] text-slate-500 font-medium">輕鬆掌握收支與每月預算</p>
                </div>
            </div>

            <!-- Month Picker & Desktop Add Button -->
            <div class="flex items-center space-x-2">
                <div class="relative flex items-center bg-slate-100 rounded-xl px-2.5 sm:px-3 py-1.5 border border-slate-200/60">
                    <i class="fa-regular fa-calendar text-indigo-600 text-xs mr-1.5 sm:mr-2"></i>
                    <input type="month" id="month-selector" onchange="onMonthChange()" class="bg-transparent text-xs sm:text-sm font-bold text-slate-700 focus:outline-none cursor-pointer">
                </div>
                <button onclick="openExportModal()" class="bg-emerald-50 hover:bg-emerald-100 active:scale-95 text-emerald-700 border border-emerald-200/80 px-3 sm:px-3.5 py-2 rounded-xl font-bold text-xs sm:text-sm transition flex items-center space-x-1.5" title="匯出 CSV 報表">
                    <i class="fa-solid fa-file-csv text-emerald-600"></i>
                    <span>匯出 CSV</span>
                </button>
                <button onclick="openModal()" class="hidden sm:flex bg-indigo-600 hover:bg-indigo-700 active:scale-95 text-white px-4 py-2 rounded-xl font-semibold text-sm transition shadow-sm items-center space-x-1.5">
                    <i class="fa-solid fa-plus"></i>
                    <span>記一筆</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="max-w-4xl mx-auto px-4 py-5 flex-grow w-full space-y-5">

        <!-- Overspending Alert Banner (Hidden by default) -->
        <div id="budget-alert-banner" class="hidden rounded-2xl p-4 border shadow-xs flex items-start space-x-3 transition-all">
            <div id="alert-icon-box" class="w-9 h-9 rounded-xl flex items-center justify-center flex-shrink-0 mt-0.5">
                <i id="alert-icon" class="fa-solid fa-triangle-exclamation"></i>
            </div>
            <div class="flex-grow">
                <h4 id="alert-title" class="text-sm font-bold">預算提醒</h4>
                <p id="alert-desc" class="text-xs mt-0.5 leading-relaxed"></p>
            </div>
        </div>

        <!-- Monthly Summary Cards (3 Columns on Desktop, Stacked/Grid on Mobile) -->
        <section class="grid grid-cols-3 gap-2.5 sm:gap-4">
            <!-- Monthly Income -->
            <div class="bg-white p-3.5 sm:p-5 rounded-2xl border border-slate-100 shadow-xs flex flex-col justify-between">
                <div class="flex items-center justify-between mb-1.5">
                    <span class="text-[11px] sm:text-xs font-semibold text-slate-400">本月收入</span>
                    <span class="w-6 h-6 sm:w-8 sm:h-8 rounded-lg bg-emerald-50 text-emerald-600 flex items-center justify-center text-xs sm:text-sm">
                        <i class="fa-solid fa-arrow-trend-up"></i>
                    </span>
                </div>
                <h2 id="total-income" class="text-base sm:text-2xl font-extrabold text-emerald-600 truncate">$0</h2>
            </div>

            <!-- Monthly Expense -->
            <div class="bg-white p-3.5 sm:p-5 rounded-2xl border border-slate-100 shadow-xs flex flex-col justify-between">
                <div class="flex items-center justify-between mb-1.5">
                    <span class="text-[11px] sm:text-xs font-semibold text-slate-400">本月支出</span>
                    <span class="w-6 h-6 sm:w-8 sm:h-8 rounded-lg bg-rose-50 text-rose-600 flex items-center justify-center text-xs sm:text-sm">
                        <i class="fa-solid fa-arrow-trend-down"></i>
                    </span>
                </div>
                <h2 id="total-expense" class="text-base sm:text-2xl font-extrabold text-rose-600 truncate">$0</h2>
            </div>

            <!-- Net Balance -->
            <div class="bg-white p-3.5 sm:p-5 rounded-2xl border border-slate-100 shadow-xs flex flex-col justify-between">
                <div class="flex items-center justify-between mb-1.5">
                    <span class="text-[11px] sm:text-xs font-semibold text-slate-400">本月結餘</span>
                    <span class="w-6 h-6 sm:w-8 sm:h-8 rounded-lg bg-indigo-50 text-indigo-600 flex items-center justify-center text-xs sm:text-sm">
                        <i class="fa-solid fa-scale-balanced"></i>
                    </span>
                </div>
                <h2 id="total-balance" class="text-base sm:text-2xl font-extrabold text-slate-900 truncate">$0</h2>
            </div>
        </section>

        <!-- Monthly Budget & Category Chart Grid -->
        <section class="grid grid-cols-1 md:grid-cols-2 gap-5">
            <!-- Monthly Budget Card -->
            <div id="budget-card" class="bg-white p-5 rounded-2xl border border-slate-100 shadow-xs flex flex-col justify-between space-y-4">
                <div class="flex justify-between items-start">
                    <div>
                        <h3 class="font-bold text-slate-800 text-sm sm:text-base flex items-center space-x-2">
                            <i class="fa-solid fa-bullseye text-indigo-500"></i>
                            <span>每月支出預算</span>
                        </h3>
                        <p class="text-xs text-slate-400 mt-0.5">設定每月上限，超支自動提醒</p>
                    </div>
                    <button onclick="openBudgetModal()" class="text-xs font-semibold bg-indigo-50 hover:bg-indigo-100 text-indigo-600 px-3 py-1.5 rounded-xl transition flex items-center space-x-1">
                        <i class="fa-solid fa-pen text-[10px]"></i>
                        <span>設定預算</span>
                    </button>
                </div>

                <!-- Budget Progress Display -->
                <div id="budget-configured-view" class="space-y-2.5 my-auto">
                    <div class="flex justify-between items-end">
                        <div>
                            <span class="text-xs text-slate-400 block">已使用 / 預算上限</span>
                            <div class="text-sm sm:text-base font-bold text-slate-800 mt-0.5">
                                <span id="budget-spent-text" class="text-rose-600">$0</span>
                                <span class="text-slate-400 font-normal mx-1">/</span>
                                <span id="budget-limit-text">$0</span>
                            </div>
                        </div>
                        <div class="text-right">
                            <span id="budget-percent-badge" class="inline-block px-2.5 py-1 rounded-lg text-xs font-bold bg-emerald-50 text-emerald-600">0%</span>
                        </div>
                    </div>

                    <!-- Progress Bar -->
                    <div class="w-full h-3.5 bg-slate-100 rounded-full overflow-hidden p-0.5">
                        <div id="budget-progress-bar" class="h-full rounded-full bg-indigo-600 transition-all duration-500" style="width: 0%"></div>
                    </div>

                    <div class="flex justify-between text-xs text-slate-500 pt-0.5">
                        <span id="budget-remaining-text">剩餘可用額度：$0</span>
                        <span id="budget-status-label" class="font-medium text-emerald-600">預算控制良好</span>
                    </div>
                </div>

                <!-- Unconfigured Budget Prompt -->
                <div id="budget-empty-view" class="hidden py-6 text-center bg-slate-50 rounded-xl border border-dashed border-slate-200">
                    <p class="text-xs text-slate-500 mb-2.5">尚未設定每月支出預算上限</p>
                    <button onclick="openBudgetModal()" class="bg-white border border-slate-200 hover:border-indigo-300 text-indigo-600 text-xs font-semibold px-4 py-2 rounded-xl shadow-2xs transition">
                        立即設定預算金額
                    </button>
                </div>
            </div>

            <!-- Category Pie/Doughnut Chart Card -->
            <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-xs flex flex-col justify-between">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="font-bold text-slate-800 text-sm sm:text-base flex items-center space-x-2">
                        <i class="fa-solid fa-chart-pie text-indigo-500"></i>
                        <span>本月支出分類圓餅圖</span>
                    </h3>
                    <span id="chart-month-badge" class="text-[11px] font-semibold text-slate-500 bg-slate-100 px-2.5 py-1 rounded-lg">2026-11</span>
                </div>
                <div class="relative w-full h-48 sm:h-52 flex items-center justify-center">
                    <canvas id="expenseChart"></canvas>
                </div>
            </div>
        </section>

        <!-- Transaction History Section -->
        <section class="bg-white rounded-2xl border border-slate-100 shadow-xs overflow-hidden">
            <div class="p-4 sm:p-5 border-b border-slate-100 flex flex-wrap justify-between items-center gap-3">
                <div class="flex items-center space-x-2">
                    <i class="fa-solid fa-receipt text-indigo-500"></i>
                    <h3 class="font-bold text-slate-800 text-sm sm:text-base">本月收支明細</h3>
                    <span id="tx-count-badge" class="text-xs bg-slate-100 text-slate-600 font-bold px-2 py-0.5 rounded-full">0</span>
                </div>

                <!-- Filter Tabs & Data Actions -->
                <div class="flex items-center space-x-2">
                    <div class="bg-slate-100 p-1 rounded-xl flex space-x-1 text-xs">
                        <button onclick="setFilter('all')" id="filter-all" class="px-2.5 py-1 rounded-lg font-semibold transition bg-white text-indigo-600 shadow-2xs">全部</button>
                        <button onclick="setFilter('expense')" id="filter-expense" class="px-2.5 py-1 rounded-lg font-semibold text-slate-600 transition hover:text-slate-900">支出</button>
                        <button onclick="setFilter('income')" id="filter-income" class="px-2.5 py-1 rounded-lg font-semibold text-slate-600 transition hover:text-slate-900">收入</button>
                    </div>
                    <button onclick="openExportModal()" class="hidden sm:flex items-center space-x-1.5 px-3 py-1.5 rounded-xl bg-emerald-50 hover:bg-emerald-100 text-emerald-700 font-bold text-xs transition border border-emerald-200/60" title="匯出 CSV">
                        <i class="fa-solid fa-download"></i>
                        <span>匯出 CSV</span>
                    </button>
                    <button onclick="openDataMenuModal()" class="w-8 h-8 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-600 flex items-center justify-center transition" title="資料管理">
                        <i class="fa-solid fa-ellipsis-vertical text-xs"></i>
                    </button>
                </div>
            </div>

            <!-- Mobile-Friendly Transaction List -->
            <div id="transaction-list" class="divide-y divide-slate-100">
                <!-- Dynamically populated -->
            </div>

            <!-- Empty State -->
            <div id="empty-state" class="hidden py-14 px-4 text-center">
                <div class="w-14 h-14 bg-slate-100 text-slate-400 rounded-2xl flex items-center justify-center mx-auto mb-3 text-xl">
                    <i class="fa-solid fa-folder-open"></i>
                </div>
                <p class="text-slate-600 font-semibold text-sm">此月份尚無收支紀錄</p>
                <p class="text-slate-400 text-xs mt-1 mb-4">點擊下方「記一筆」按鈕開始記錄您的日常開銷</p>
                <button onclick="loadSampleData()" class="text-xs text-indigo-600 hover:text-indigo-700 font-semibold bg-indigo-50 px-3.5 py-2 rounded-xl transition">
                    <i class="fa-solid fa-wand-magic-sparkles mr-1"></i> 載入範例資料體驗
                </button>
            </div>
        </section>

    </main>

    <!-- Mobile Fixed Bottom Floating Action Button (Thumb-Friendly) -->
    <div class="fixed bottom-5 inset-x-0 z-30 flex justify-center sm:hidden pointer-events-none px-4">
        <button onclick="openModal()" class="pointer-events-auto bg-indigo-600 hover:bg-indigo-700 active:scale-95 text-white px-6 py-3.5 rounded-full font-bold text-sm shadow-lg shadow-indigo-600/30 flex items-center space-x-2 transition">
            <i class="fa-solid fa-plus text-base"></i>
            <span>記一筆收支</span>
        </button>
    </div>

    <!-- Add Transaction Modal (Bottom Sheet on Mobile, Centered Card on Desktop) -->
    <div id="transaction-modal" class="fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-end sm:items-center justify-center p-0 sm:p-4 hidden opacity-0 transition-opacity duration-300">
        <div id="modal-content" class="bg-white rounded-t-3xl sm:rounded-3xl max-w-md w-full p-5 sm:p-6 shadow-2xl border border-slate-100 transform translate-y-full sm:translate-y-0 sm:scale-95 transition-transform duration-300 max-h-[92vh] overflow-y-auto">
            <!-- Drag Handle for Mobile -->
            <div class="w-10 h-1 bg-slate-200 rounded-full mx-auto mb-4 sm:hidden"></div>

            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-extrabold text-slate-900">新增收支紀錄</h3>
                <button onclick="closeModal()" class="w-8 h-8 rounded-full bg-slate-100 text-slate-500 hover:bg-slate-200 flex items-center justify-center transition">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <form id="transaction-form" onsubmit="handleFormSubmit(event)" class="space-y-4">
                <!-- Expense / Income Toggle -->
                <div class="grid grid-cols-2 gap-2 bg-slate-100 p-1 rounded-2xl">
                    <button type="button" id="type-expense-btn" onclick="switchType('expense')" class="py-2.5 rounded-xl font-bold text-sm transition bg-white text-rose-600 shadow-xs flex items-center justify-center space-x-1.5">
                        <i class="fa-solid fa-minus-circle"></i>
                        <span>支出</span>
                    </button>
                    <button type="button" id="type-income-btn" onclick="switchType('income')" class="py-2.5 rounded-xl font-bold text-sm transition text-slate-600 hover:text-slate-900 flex items-center justify-center space-x-1.5">
                        <i class="fa-solid fa-plus-circle"></i>
                        <span>收入</span>
                    </button>
                </div>

                <!-- Amount Input (Numeric Keypad Optimized) -->
                <div>
                    <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">金額 (TWD)</label>
                    <div class="relative">
                        <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-slate-400 font-extrabold text-lg">$</span>
                        <input type="number" id="amount" inputmode="decimal" required min="1" step="any" placeholder="0" class="w-full pl-9 pr-4 py-3 rounded-2xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 text-xl font-extrabold text-slate-900 bg-slate-50/50">
                    </div>
                </div>

                <!-- Category Touch Pills Grid -->
                <div>
                    <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">選擇分類</label>
                    <input type="hidden" id="category" required>
                    <div id="category-pills" class="grid grid-cols-4 gap-2">
                        <!-- Dynamically populated pills -->
                    </div>
                </div>

                <!-- Date & Note -->
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">日期</label>
                        <input type="date" id="date" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 text-sm font-medium bg-white">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">備註 (選填)</label>
                        <input type="text" id="description" placeholder="例如：午餐、加油..." class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 text-sm">
                    </div>
                </div>

                <!-- Submit Button -->
                <div class="pt-2">
                    <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 active:scale-[0.99] text-white font-bold py-3.5 rounded-2xl transition shadow-md shadow-indigo-600/20 text-sm flex items-center justify-center space-x-2">
                        <i class="fa-solid fa-check"></i>
                        <span>儲存這筆紀錄</span>
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Budget Settings Modal -->
    <div id="budget-modal" class="fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4 hidden opacity-0 transition-opacity duration-300">
        <div id="budget-modal-content" class="bg-white rounded-3xl max-w-sm w-full p-6 shadow-2xl border border-slate-100 transform scale-95 transition-transform duration-300">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-extrabold text-slate-900 flex items-center space-x-2">
                    <i class="fa-solid fa-bullseye text-indigo-600"></i>
                    <span>設定每月支出預算</span>
                </h3>
                <button onclick="closeBudgetModal()" class="w-8 h-8 rounded-full bg-slate-100 text-slate-500 hover:bg-slate-200 flex items-center justify-center transition">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>
            <p class="text-xs text-slate-500 mb-4 leading-relaxed">當本月累計支出達到預算的 <span class="font-bold text-amber-600">80%</span> 或 <span class="font-bold text-rose-600">100%</span> 時，系統將自動顯示超支警告提醒。</p>
            <form onsubmit="handleBudgetSubmit(event)" class="space-y-4">
                <div>
                    <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">每月預算金額 (TWD)</label>
                    <div class="relative">
                        <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-slate-400 font-extrabold">$</span>
                        <input type="number" id="budget-input" inputmode="decimal" min="0" step="any" placeholder="例如：20000 (設為 0 則關閉)" class="w-full pl-9 pr-4 py-3 rounded-2xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 text-lg font-extrabold text-slate-900">
                    </div>
                </div>
                <div class="flex space-x-2 pt-1">
                    <button type="button" onclick="closeBudgetModal()" class="w-1/3 bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold py-3 rounded-xl text-sm transition">取消</button>
                    <button type="submit" class="w-2/3 bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 rounded-xl text-sm transition shadow-md shadow-indigo-600/20">儲存預算設定</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Data Management Modal -->
    <div id="data-modal" class="fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4 hidden opacity-0 transition-opacity duration-300">
        <div id="data-modal-content" class="bg-white rounded-3xl max-w-sm w-full p-6 shadow-2xl border border-slate-100 transform scale-95 transition-transform duration-300">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-base font-extrabold text-slate-900">資料管理與備份</h3>
                <button onclick="closeDataMenuModal()" class="w-8 h-8 rounded-full bg-slate-100 text-slate-500 hover:bg-slate-200 flex items-center justify-center transition">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>
            <div class="space-y-2.5">
                <button onclick="closeDataMenuModal(); openExportModal();" class="w-full flex items-center justify-between p-3.5 rounded-2xl bg-slate-50 hover:bg-indigo-50 text-slate-700 hover:text-indigo-700 font-semibold text-sm transition border border-slate-100">
                    <span class="flex items-center space-x-2.5">
                        <i class="fa-solid fa-file-csv text-indigo-600"></i>
                        <span>匯出 CSV 報表</span>
                    </span>
                    <i class="fa-solid fa-chevron-right text-xs opacity-50"></i>
                </button>
                <button onclick="loadSampleData(true)" class="w-full flex items-center justify-between p-3.5 rounded-2xl bg-slate-50 hover:bg-emerald-50 text-slate-700 hover:text-emerald-700 font-semibold text-sm transition border border-slate-100">
                    <span class="flex items-center space-x-2.5">
                        <i class="fa-solid fa-wand-magic-sparkles text-emerald-600"></i>
                        <span>加入範例測試資料</span>
                    </span>
                    <i class="fa-solid fa-chevron-right text-xs opacity-50"></i>
                </button>
                <div id="clear-confirm-box" class="pt-2">
                    <button id="clear-init-btn" onclick="showClearConfirm()" class="w-full flex items-center justify-between p-3.5 rounded-2xl bg-rose-50/60 hover:bg-rose-50 text-rose-600 font-semibold text-sm transition border border-rose-100">
                        <span class="flex items-center space-x-2.5">
                            <i class="fa-solid fa-trash-can"></i>
                            <span>清除所有記帳資料</span>
                        </span>
                    </button>
                    <div id="clear-action-row" class="hidden bg-rose-50 p-3 rounded-2xl border border-rose-200 space-y-2">
                        <p class="text-xs text-rose-700 font-semibold text-center">確定要刪除所有紀錄嗎？此動作無法復原。</p>
                        <div class="grid grid-cols-2 gap-2">
                            <button onclick="hideClearConfirm()" class="py-2 rounded-xl bg-white text-slate-600 text-xs font-bold border border-slate-200">取消</button>
                            <button onclick="clearAllData()" class="py-2 rounded-xl bg-rose-600 text-white text-xs font-bold">確定清空</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Export CSV Options Modal -->
    <div id="export-modal" class="fixed inset-0 z-50 bg-slate-900/50 backdrop-blur-xs flex items-center justify-center p-4 hidden opacity-0 transition-opacity duration-300">
        <div id="export-modal-content" class="bg-white rounded-3xl max-w-sm w-full p-6 shadow-2xl border border-slate-100 transform scale-95 transition-transform duration-300">
            <div class="flex justify-between items-center mb-3">
                <h3 class="text-lg font-extrabold text-slate-900 flex items-center space-x-2">
                    <i class="fa-solid fa-file-csv text-emerald-600"></i>
                    <span>匯出 CSV 報表</span>
                </h3>
                <button onclick="closeExportModal()" class="w-8 h-8 rounded-full bg-slate-100 text-slate-500 hover:bg-slate-200 flex items-center justify-center transition">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>
            <p class="text-xs text-slate-500 mb-4 leading-relaxed">檔案採用 UTF-8 BOM 編碼，可直接使用 Excel 或 Google 試算表開啟不亂碼。</p>
            <div class="space-y-2.5">
                <button onclick="exportCSV('month')" class="w-full flex items-center justify-between p-4 rounded-2xl bg-emerald-50/80 hover:bg-emerald-100/80 text-emerald-900 border border-emerald-200 transition text-left">
                    <div>
                        <div class="font-bold text-sm flex items-center space-x-1.5">
                            <span>匯出本月明細 (<span id="export-month-label">2026-11</span>)</span>
                        </div>
                        <p id="export-month-count" class="text-xs text-emerald-700 mt-0.5">共 0 筆紀錄</p>
                    </div>
                    <div class="w-9 h-9 rounded-xl bg-emerald-600 text-white flex items-center justify-center flex-shrink-0">
                        <i class="fa-solid fa-download text-xs"></i>
                    </div>
                </button>

                <button onclick="exportCSV('all')" class="w-full flex items-center justify-between p-4 rounded-2xl bg-slate-50 hover:bg-indigo-50/70 text-slate-800 border border-slate-200/80 transition text-left">
                    <div>
                        <div class="font-bold text-sm">匯出全部歷史明細</div>
                        <p id="export-all-count" class="text-xs text-slate-500 mt-0.5">共 0 筆紀錄</p>
                    </div>
                    <div class="w-9 h-9 rounded-xl bg-slate-200 text-slate-700 flex items-center justify-center flex-shrink-0">
                        <i class="fa-solid fa-box-archive text-xs"></i>
                    </div>
                </button>
            </div>
        </div>
    </div>

    <!-- JavaScript Application Logic -->
    <script>
        // Storage Keys
        const STORAGE_TX_KEY = 'smart_expense_tracker_tx_v2';
        const STORAGE_BUDGET_KEY = 'smart_expense_tracker_budget_v2';

        // Preset Categories with Icons & Colors
        const categories = {
            expense: [
                { name: '餐飲', icon: 'fa-utensils', color: 'bg-amber-100 text-amber-700', hex: '#f59e0b' },
                { name: '交通', icon: 'fa-train-subway', color: 'bg-cyan-100 text-cyan-700', hex: '#06b6d4' },
                { name: '購物', icon: 'fa-bag-shopping', color: 'bg-purple-100 text-purple-700', hex: '#a855f7' },
                { name: '娛樂', icon: 'fa-gamepad', color: 'bg-pink-100 text-pink-700', hex: '#ec4899' },
                { name: '居住', icon: 'fa-house', color: 'bg-blue-100 text-blue-700', hex: '#3b82f6' },
                { name: '醫療', icon: 'fa-kit-medical', color: 'bg-rose-100 text-rose-700', hex: '#f43f5e' },
                { name: '學習', icon: 'fa-book-open', color: 'bg-emerald-100 text-emerald-700', hex: '#10b981' },
                { name: '其他', icon: 'fa-tag', color: 'bg-slate-100 text-slate-700', hex: '#64748b' }
            ],
            income: [
                { name: '薪資', icon: 'fa-money-bill-wave', color: 'bg-emerald-100 text-emerald-700', hex: '#10b981' },
                { name: '獎金', icon: 'fa-award', color: 'bg-amber-100 text-amber-700', hex: '#f59e0b' },
                { name: '投資', icon: 'fa-chart-line', color: 'bg-indigo-100 text-indigo-700', hex: '#6366f1' },
                { name: '其他', icon: 'fa-wallet', color: 'bg-slate-100 text-slate-700', hex: '#64748b' }
            ]
        };

        // Get Local YYYY-MM and YYYY-MM-DD Strings
        function getTodayStr() {
            const now = new Date();
            const y = now.getFullYear();
            const m = String(now.getMonth() + 1).padStart(2, '0');
            const d = String(now.getDate()).padStart(2, '0');
            return `${y}-${m}-${d}`;
        }

        function getCurrentMonthStr() {
            return getTodayStr().slice(0, 7);
        }

        // Application State
        let transactions = [];
        let monthlyBudget = 0;
        let selectedMonth = getCurrentMonthStr();
        let currentType = 'expense';
        let selectedCategory = '餐飲';
        let currentFilter = 'all';
        let expenseChartInstance = null;

        // Initialize App
        document.addEventListener('DOMContentLoaded', () => {
            // Load from localStorage
            try {
                const savedTx = localStorage.getItem(STORAGE_TX_KEY);
                transactions = savedTx ? JSON.parse(savedTx) : [];
                const savedBudget = localStorage.getItem(STORAGE_BUDGET_KEY);
                monthlyBudget = savedBudget ? parseFloat(savedBudget) : 15000; // Default helpful budget of $15,000
            } catch (err) {
                console.error('Failed to read localStorage:', err);
                transactions = [];
                monthlyBudget = 15000;
            }

            // Set month picker to current month
            const monthInput = document.getElementById('month-selector');
            monthInput.value = selectedMonth;

            // Initialize form defaults
            document.getElementById('date').value = getTodayStr();
            renderCategoryPills();
            renderApp();
        });

        // Month Selection Changed
        function onMonthChange() {
            const val = document.getElementById('month-selector').value;
            if (val) {
                selectedMonth = val;
                renderApp();
            }
        }

        // Get Transactions for Selected Month
        function getMonthTransactions() {
            return transactions
                .filter(t => t.date && t.date.startsWith(selectedMonth))
                .sort((a, b) => new Date(b.date) - new Date(a.date) || Number(b.id) - Number(a.id));
        }

        // --- Modal Controls: Add Transaction ---
        function openModal() {
            const modal = document.getElementById('transaction-modal');
            const content = document.getElementById('modal-content');
            
            // Set date input within selected month if needed, else today
            const today = getTodayStr();
            document.getElementById('date').value = today.startsWith(selectedMonth) ? today : `${selectedMonth}-01`;
            
            modal.classList.remove('hidden');
            requestAnimationFrame(() => {
                modal.classList.remove('opacity-0');
                content.classList.remove('translate-y-full', 'sm:scale-95');
            });
            setTimeout(() => document.getElementById('amount').focus(), 150);
        }

        function closeModal() {
            const modal = document.getElementById('transaction-modal');
            const content = document.getElementById('modal-content');
            modal.classList.add('opacity-0');
            content.classList.add('translate-y-full', 'sm:scale-95');
            setTimeout(() => {
                modal.classList.add('hidden');
                document.getElementById('transaction-form').reset();
                switchType('expense');
            }, 280);
        }

        // Switch Expense / Income in Form
        function switchType(type) {
            currentType = type;
            const expBtn = document.getElementById('type-expense-btn');
            const incBtn = document.getElementById('type-income-btn');

            if (type === 'expense') {
                expBtn.className = "py-2.5 rounded-xl font-bold text-sm transition bg-white text-rose-600 shadow-xs flex items-center justify-center space-x-1.5";
                incBtn.className = "py-2.5 rounded-xl font-bold text-sm transition text-slate-600 hover:text-slate-900 flex items-center justify-center space-x-1.5";
                selectedCategory = categories.expense[0].name;
            } else {
                incBtn.className = "py-2.5 rounded-xl font-bold text-sm transition bg-white text-emerald-600 shadow-xs flex items-center justify-center space-x-1.5";
                expBtn.className = "py-2.5 rounded-xl font-bold text-sm transition text-slate-600 hover:text-slate-900 flex items-center justify-center space-x-1.5";
                selectedCategory = categories.income[0].name;
            }
            renderCategoryPills();
        }

        // Render Interactive Category Pills
        function renderCategoryPills() {
            const container = document.getElementById('category-pills');
            const hiddenInput = document.getElementById('category');
            container.innerHTML = '';

            const list = categories[currentType];
            if (!list.some(c => c.name === selectedCategory)) {
                selectedCategory = list[0].name;
            }
            hiddenInput.value = selectedCategory;

            list.forEach(cat => {
                const isSelected = cat.name === selectedCategory;
                const btn = document.createElement('button');
                btn.type = 'button';
                btn.onclick = () => {
                    selectedCategory = cat.name;
                    renderCategoryPills();
                };
                btn.className = `flex flex-col items-center justify-center p-2.5 rounded-2xl border transition ${
                    isSelected 
                        ? 'border-indigo-600 bg-indigo-50/70 text-indigo-700 shadow-2xs ring-2 ring-indigo-500/10' 
                        : 'border-slate-100 bg-slate-50/70 text-slate-600 hover:bg-slate-100'
                }`;
                btn.innerHTML = `
                    <div class="w-8 h-8 rounded-xl ${cat.color} flex items-center justify-center text-xs mb-1">
                        <i class="fa-solid ${cat.icon}"></i>
                    </div>
                    <span class="text-xs font-bold">${cat.name}</span>
                `;
                container.appendChild(btn);
            });
        }

        // Handle Add Transaction Submit
        function handleFormSubmit(e) {
            e.preventDefault();
            const amount = parseFloat(document.getElementById('amount').value);
            const category = document.getElementById('category').value;
            const date = document.getElementById('date').value;
            const description = document.getElementById('description').value.trim();

            if (!amount || isNaN(amount) || amount <= 0) {
                showToast('請輸入大於 0 的有效金額', 'error');
                return;
            }

            const newTx = {
                id: Date.now().toString(),
                type: currentType,
                amount: amount,
                category: category,
                date: date,
                description: description
            };

            transactions.unshift(newTx);
            
            // Switch view to the transaction's month so user sees it immediately
            if (date && !date.startsWith(selectedMonth)) {
                selectedMonth = date.slice(0, 7);
                document.getElementById('month-selector').value = selectedMonth;
            }

            saveTransactions();
            closeModal();
            renderApp();

            // Check if this expense triggered overspending
            if (newTx.type === 'expense' && monthlyBudget > 0) {
                const monthExpense = getMonthTransactions()
                    .filter(t => t.type === 'expense')
                    .reduce((sum, t) => sum + t.amount, 0);
                if (monthExpense > monthlyBudget) {
                    showToast(`警告！本月支出已超過預算上限！`, 'error');
                    return;
                }
            }
            showToast('已成功新增一筆紀錄');
        }

        // Delete Transaction
        function deleteTransaction(id) {
            transactions = transactions.filter(t => t.id !== id);
            saveTransactions();
            renderApp();
            showToast('已刪除該筆紀錄');
        }

        // --- Budget Modal Controls ---
        function openBudgetModal() {
            const modal = document.getElementById('budget-modal');
            const content = document.getElementById('budget-modal-content');
            document.getElementById('budget-input').value = monthlyBudget > 0 ? monthlyBudget : '';
            modal.classList.remove('hidden');
            requestAnimationFrame(() => {
                modal.classList.remove('opacity-0');
                content.classList.remove('scale-95');
            });
            setTimeout(() => document.getElementById('budget-input').focus(), 150);
        }

        function closeBudgetModal() {
            const modal = document.getElementById('budget-modal');
            const content = document.getElementById('budget-modal-content');
            modal.classList.add('opacity-0');
            content.classList.add('scale-95');
            setTimeout(() => modal.classList.add('hidden'), 280);
        }

        function handleBudgetSubmit(e) {
            e.preventDefault();
            const val = parseFloat(document.getElementById('budget-input').value);
            monthlyBudget = (!isNaN(val) && val > 0) ? val : 0;
            localStorage.setItem(STORAGE_BUDGET_KEY, monthlyBudget.toString());
            closeBudgetModal();
            renderApp();
            showToast(monthlyBudget > 0 ? `每月預算已設為 ${formatCurrency(monthlyBudget)}` : '已關閉每月預算限制');
        }

        // --- Data Management Modal Controls ---
        function openDataMenuModal() {
            hideClearConfirm();
            const modal = document.getElementById('data-modal');
            const content = document.getElementById('data-modal-content');
            modal.classList.remove('hidden');
            requestAnimationFrame(() => {
                modal.classList.remove('opacity-0');
                content.classList.remove('scale-95');
            });
        }

        function closeDataMenuModal() {
            const modal = document.getElementById('data-modal');
            const content = document.getElementById('data-modal-content');
            modal.classList.add('opacity-0');
            content.classList.add('scale-95');
            setTimeout(() => modal.classList.add('hidden'), 280);
        }

        function showClearConfirm() {
            document.getElementById('clear-init-btn').classList.add('hidden');
            document.getElementById('clear-action-row').classList.remove('hidden');
        }

        function hideClearConfirm() {
            document.getElementById('clear-init-btn').classList.remove('hidden');
            document.getElementById('clear-action-row').classList.add('hidden');
        }

        function clearAllData() {
            transactions = [];
            saveTransactions();
            closeDataMenuModal();
            renderApp();
            showToast('已清空所有記帳資料');
        }

        // --- Export CSV Modal & Logic ---
        function openExportModal() {
            const monthTx = getMonthTransactions();
            document.getElementById('export-month-label').textContent = selectedMonth;
            document.getElementById('export-month-count').textContent = `共 ${monthTx.length} 筆紀錄`;
            document.getElementById('export-all-count').textContent = `共 ${transactions.length} 筆紀錄`;

            const modal = document.getElementById('export-modal');
            const content = document.getElementById('export-modal-content');
            modal.classList.remove('hidden');
            requestAnimationFrame(() => {
                modal.classList.remove('opacity-0');
                content.classList.remove('scale-95');
            });
        }

        function closeExportModal() {
            const modal = document.getElementById('export-modal');
            const content = document.getElementById('export-modal-content');
            modal.classList.add('opacity-0');
            content.classList.add('scale-95');
            setTimeout(() => modal.classList.add('hidden'), 280);
        }

        function escapeCsvField(value) {
            const str = String(value ?? '');
            if (/[",\r\n]/.test(str)) {
                return `"${str.replace(/"/g, '""')}"`;
            }
            return str;
        }

        function exportCSV(scope = 'month') {
            const targetTx = scope === 'all'
                ? [...transactions].sort((a, b) => new Date(b.date) - new Date(a.date) || Number(b.id) - Number(a.id))
                : getMonthTransactions();

            if (targetTx.length === 0) {
                showToast(scope === 'all' ? '目前沒有任何資料可匯出' : `${selectedMonth} 沒有資料可匯出`, 'error');
                return;
            }

            const headers = ['日期', '類型', '分類', '金額', '備註'];
            const rows = targetTx.map(t => [
                escapeCsvField(t.date),
                escapeCsvField(t.type === 'income' ? '收入' : '支出'),
                escapeCsvField(t.category),
                escapeCsvField(t.amount),
                escapeCsvField(t.description || '')
            ]);

            const csvContent = '\uFEFF' + [headers.join(','), ...rows.map(r => r.join(','))].join('\r\n');
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const filename = scope === 'all' ? `記帳明細_全部紀錄.csv` : `記帳明細_${selectedMonth}.csv`;

            const a = document.createElement('a');
            a.href = url;
            a.download = filename;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);

            closeExportModal();
            showToast(`已匯出 ${filename}`);
        }

        function loadSampleData(fromModal = false) {
            const m = selectedMonth;
            const samples = [
                { id: (Date.now() + 1).toString(), type: 'income', amount: 48000, category: '薪資', date: `${m}-05`, description: '本月薪資入帳' },
                { id: (Date.now() + 2).toString(), type: 'expense', amount: 4500, category: '居住', date: `${m}-06`, description: '水電瓦斯與網路費' },
                { id: (Date.now() + 3).toString(), type: 'expense', amount: 3200, category: '餐飲', date: `${m}-10`, description: '超市採買與聚餐' },
                { id: (Date.now() + 4).toString(), type: 'expense', amount: 1280, category: '交通', date: `${m}-12`, description: '通勤月票儲值' },
                { id: (Date.now() + 5).toString(), type: 'expense', amount: 2650, category: '購物', date: `${m}-15`, description: '日用品與服飾' },
                { id: (Date.now() + 6).toString(), type: 'expense', amount: 980, category: '娛樂', date: `${m}-17`, description: '週末電影與咖啡' }
            ];
            transactions = [...samples, ...transactions];
            saveTransactions();
            if (fromModal) closeDataMenuModal();
            renderApp();
            showToast('已載入本月範例資料');
        }

        function saveTransactions() {
            localStorage.setItem(STORAGE_TX_KEY, JSON.stringify(transactions));
        }

        // Filter Controls
        function setFilter(filter) {
            currentFilter = filter;
            ['all', 'expense', 'income'].forEach(f => {
                const btn = document.getElementById(`filter-${f}`);
                if (f === filter) {
                    btn.className = "px-2.5 py-1 rounded-lg font-semibold transition bg-white text-indigo-600 shadow-2xs";
                } else {
                    btn.className = "px-2.5 py-1 rounded-lg font-semibold text-slate-600 transition hover:text-slate-900";
                }
            });
            renderTransactionsList();
        }

        // --- Main Render Pipeline ---
        function renderApp() {
            const monthTx = getMonthTransactions();
            let income = 0;
            let expense = 0;

            monthTx.forEach(t => {
                if (t.type === 'income') income += t.amount;
                if (t.type === 'expense') expense += t.amount;
            });

            const balance = income - expense;

            // Update Summary Numbers
            document.getElementById('total-income').textContent = formatCurrency(income);
            document.getElementById('total-expense').textContent = formatCurrency(expense);
            const balEl = document.getElementById('total-balance');
            balEl.textContent = (balance < 0 ? '-' : '') + formatCurrency(Math.abs(balance));
            balEl.className = `text-base sm:text-2xl font-extrabold truncate ${balance < 0 ? 'text-rose-600' : 'text-slate-900'}`;

            // Update Month Badge
            document.getElementById('chart-month-badge').textContent = selectedMonth;

            // Update Budget Section & Overspend Alerts
            renderBudgetStatus(expense);

            // Render Transaction List
            renderTransactionsList();

            // Render Pie/Doughnut Chart
            renderChart(monthTx);
        }

        // Render Budget Progress Bar & Overspending Alert Banner
        function renderBudgetStatus(monthExpense) {
            const configuredView = document.getElementById('budget-configured-view');
            const emptyView = document.getElementById('budget-empty-view');
            const alertBanner = document.getElementById('budget-alert-banner');
            const budgetCard = document.getElementById('budget-card');

            if (!monthlyBudget || monthlyBudget <= 0) {
                configuredView.classList.add('hidden');
                emptyView.classList.remove('hidden');
                alertBanner.classList.add('hidden');
                budgetCard.classList.remove('overspend-pulse', 'border-rose-300');
                return;
            }

            configuredView.classList.remove('hidden');
            emptyView.classList.add('hidden');

            const percent = Math.round((monthExpense / monthlyBudget) * 100);
            const clampedWidth = Math.min(percent, 100);
            const remaining = monthlyBudget - monthExpense;

            document.getElementById('budget-spent-text').textContent = formatCurrency(monthExpense);
            document.getElementById('budget-limit-text').textContent = formatCurrency(monthlyBudget);

            const badge = document.getElementById('budget-percent-badge');
            const bar = document.getElementById('budget-progress-bar');
            const remText = document.getElementById('budget-remaining-text');
            const statusLabel = document.getElementById('budget-status-label');

            badge.textContent = `${percent}%`;
            bar.style.width = `${clampedWidth}%`;

            // Reset styles
            budgetCard.classList.remove('overspend-pulse', 'border-rose-300');

            if (monthExpense > monthlyBudget) {
                // OVERSPENT (> 100%)
                const overAmount = monthExpense - monthlyBudget;
                badge.className = "inline-block px-2.5 py-1 rounded-lg text-xs font-extrabold bg-rose-100 text-rose-700";
                bar.className = "h-full rounded-full bg-rose-600 transition-all duration-500";
                remText.innerHTML = `<span class="text-rose-600 font-bold">已超支 ${formatCurrency(overAmount)}</span>`;
                statusLabel.textContent = "已超過每月預算！";
                statusLabel.className = "font-bold text-rose-600";
                budgetCard.classList.add('overspend-pulse', 'border-rose-300');

                // Show Red Alert Banner
                alertBanner.className = "rounded-2xl p-4 border border-rose-200 bg-rose-50 text-rose-900 shadow-xs flex items-start space-x-3";
                document.getElementById('alert-icon-box').className = "w-9 h-9 rounded-xl bg-rose-600 text-white flex items-center justify-center flex-shrink-0 mt-0.5";
                document.getElementById('alert-icon').className = "fa-solid fa-circle-exclamation";
                document.getElementById('alert-title').textContent = `超支警告！本月支出已達預算的 ${percent}%`;
                document.getElementById('alert-desc').textContent = `您本月的支出 (${formatCurrency(monthExpense)}) 已超過設定預算 (${formatCurrency(monthlyBudget)}) 共 ${formatCurrency(overAmount)}，建議減少非必要開銷。`;
            } else if (percent >= 80) {
                // WARNING (80% - 100%)
                badge.className = "inline-block px-2.5 py-1 rounded-lg text-xs font-extrabold bg-amber-100 text-amber-700";
                bar.className = "h-full rounded-full bg-amber-500 transition-all duration-500";
                remText.textContent = `剩餘可用額度：${formatCurrency(remaining)}`;
                statusLabel.textContent = "接近預算上限";
                statusLabel.className = "font-bold text-amber-600";

                // Show Amber Warning Banner
                alertBanner.className = "rounded-2xl p-4 border border-amber-200 bg-amber-50 text-amber-900 shadow-xs flex items-start space-x-3";
                document.getElementById('alert-icon-box').className = "w-9 h-9 rounded-xl bg-amber-500 text-white flex items-center justify-center flex-shrink-0 mt-0.5";
                document.getElementById('alert-icon').className = "fa-solid fa-triangle-exclamation";
                document.getElementById('alert-title').textContent = `預算即將用罄（已使用 ${percent}%）`;
                document.getElementById('alert-desc').textContent = `本月剩餘可用預算僅剩 ${formatCurrency(remaining)}，請留意接下來的消費支出。`;
            } else {
                // HEALTHY (< 80%)
                badge.className = "inline-block px-2.5 py-1 rounded-lg text-xs font-bold bg-emerald-50 text-emerald-600";
                bar.className = "h-full rounded-full bg-indigo-600 transition-all duration-500";
                remText.textContent = `剩餘可用額度：${formatCurrency(remaining)}`;
                statusLabel.textContent = "預算控制良好";
                statusLabel.className = "font-semibold text-emerald-600";
                alertBanner.classList.add('hidden');
            }
        }

        // Render Transaction History Cards
        function renderTransactionsList() {
            const listEl = document.getElementById('transaction-list');
            const emptyState = document.getElementById('empty-state');
            const countBadge = document.getElementById('tx-count-badge');

            let filtered = getMonthTransactions();
            if (currentFilter !== 'all') {
                filtered = filtered.filter(t => t.type === currentFilter);
            }

            countBadge.textContent = filtered.length;
            listEl.innerHTML = '';

            if (filtered.length === 0) {
                emptyState.classList.remove('hidden');
                return;
            }
            emptyState.classList.add('hidden');

            filtered.forEach(t => {
                const catObj = [...categories.expense, ...categories.income].find(c => c.name === t.category)
                    || { icon: 'fa-tag', color: 'bg-slate-100 text-slate-700' };

                const isIncome = t.type === 'income';
                const amountText = (isIncome ? '+' : '-') + formatCurrency(t.amount);
                const amountClass = isIncome ? 'text-emerald-600' : 'text-rose-600';
                const noteText = t.description ? escapeHtml(t.description) : t.category;

                const row = document.createElement('div');
                row.className = "p-4 sm:px-5 hover:bg-slate-50/80 transition flex items-center justify-between gap-3";
                row.innerHTML = `
                    <div class="flex items-center space-x-3 min-w-0">
                        <div class="w-10 h-10 rounded-2xl ${catObj.color} flex items-center justify-center text-sm flex-shrink-0">
                            <i class="fa-solid ${catObj.icon}"></i>
                        </div>
                        <div class="min-w-0">
                            <div class="flex items-center space-x-2">
                                <span class="font-bold text-slate-900 text-sm truncate">${noteText}</span>
                                <span class="text-[11px] font-semibold px-2 py-0.5 rounded-md bg-slate-100 text-slate-500 flex-shrink-0">${t.category}</span>
                            </div>
                            <p class="text-xs text-slate-400 mt-0.5">${t.date}</p>
                        </div>
                    </div>
                    <div class="flex items-center space-x-2.5 flex-shrink-0">
                        <span class="font-extrabold text-sm sm:text-base ${amountClass}">${amountText}</span>
                        <button onclick="deleteTransaction('${t.id}')" class="w-8 h-8 rounded-xl text-slate-300 hover:text-rose-600 hover:bg-rose-50 active:bg-rose-100 transition flex items-center justify-center" title="刪除此筆">
                            <i class="fa-regular fa-trash-can text-xs"></i>
                        </button>
                    </div>
                `;
                listEl.appendChild(row);
            });
        }

        // Render Category Pie / Doughnut Chart
        function renderChart(monthTx) {
            const ctx = document.getElementById('expenseChart').getContext('2d');

            // Group expenses by category
            const totals = {};
            monthTx.forEach(t => {
                if (t.type === 'expense') {
                    totals[t.category] = (totals[t.category] || 0) + t.amount;
                }
            });

            const entries = Object.entries(totals).filter(([, val]) => val > 0).sort((a, b) => b[1] - a[1]);

            if (expenseChartInstance) {
                expenseChartInstance.destroy();
            }

            if (entries.length === 0) {
                expenseChartInstance = new Chart(ctx, {
                    type: 'doughnut',
                    data: {
                        labels: ['本月尚無支出'],
                        datasets: [{
                            data: [1],
                            backgroundColor: ['#f1f5f9'],
                            borderWidth: 0
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        cutout: '68%',
                        plugins: {
                            tooltip: { enabled: false },
                            legend: {
                                position: 'right',
                                labels: { boxWidth: 10, font: { family: 'Inter', size: 12 }, color: '#94a3b8' }
                            }
                        }
                    }
                });
                return;
            }

            const totalExpense = entries.reduce((sum, [, val]) => sum + val, 0);
            const labels = entries.map(([name, val]) => {
                const pct = Math.round((val / totalExpense) * 100);
                return `${name} (${pct}%)`;
            });
            const data = entries.map(([, val]) => val);
            const colors = entries.map(([name]) => {
                const found = categories.expense.find(c => c.name === name);
                return found ? found.hex : '#64748b';
            });

            expenseChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: colors,
                        borderWidth: 2,
                        borderColor: '#ffffff',
                        hoverOffset: 6
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    cutout: '62%',
                    plugins: {
                        legend: {
                            position: 'right',
                            labels: {
                                boxWidth: 10,
                                padding: 12,
                                usePointStyle: true,
                                pointStyle: 'circle',
                                font: { family: 'Inter', size: 11, weight: '600' },
                                color: '#334155'
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const val = context.raw || 0;
                                    return ` $${val.toLocaleString('en-US')}`;
                                }
                            }
                        }
                    }
                }
            });
        }

        // Formatting & Helpers
        function formatCurrency(amount) {
            return '$' + Number(amount).toLocaleString('en-US', { minimumFractionDigits: 0, maximumFractionDigits: 2 });
        }

        function escapeHtml(text) {
            const map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' };
            return String(text).replace(/[&<>"']/g, m => map[m]);
        }

        let toastTimer = null;
        function showToast(message, type = 'success') {
            const toast = document.getElementById('toast');
            const icon = document.getElementById('toast-icon');
            const msg = document.getElementById('toast-message');

            msg.textContent = message;
            if (type === 'error') {
                toast.className = "fixed top-4 left-1/2 -translate-x-1/2 sm:left-auto sm:right-5 sm:translate-x-0 z-50 transition-transform duration-300 bg-rose-600 text-white px-5 py-3 rounded-2xl shadow-xl flex items-center space-x-2.5 max-w-[90vw]";
                icon.className = "fa-solid fa-circle-exclamation text-white";
            } else {
                toast.className = "fixed top-4 left-1/2 -translate-x-1/2 sm:left-auto sm:right-5 sm:translate-x-0 z-50 transition-transform duration-300 bg-slate-900 text-white px-5 py-3 rounded-2xl shadow-xl flex items-center space-x-2.5 max-w-[90vw]";
                icon.className = "fa-solid fa-circle-check text-emerald-400";
            }

            if (toastTimer) clearTimeout(toastTimer);
            toastTimer = setTimeout(() => {
                toast.classList.add('-translate-y-24', 'sm:translate-y-0', 'sm:translate-x-[150%]');
            }, 3000);
        }
    </script>
</body>
</html>
