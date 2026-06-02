<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, viewport-fit=cover">
    <title>欠款现金看板 · 云同步版</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>
    <!-- Firebase SDK (compat 版本) -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Inter', system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif; }
        body { background: #f0f4f9; padding: 16px 12px; color: #1a2c3e; }
        .password-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.7); backdrop-filter: blur(6px); display: flex; align-items: center; justify-content: center; z-index: 2000; }
        .password-modal { background: white; border-radius: 40px; padding: 32px 24px; width: 300px; text-align: center; }
        .password-modal i { font-size: 48px; color: #1f5e7e; margin-bottom: 16px; }
        .password-modal input { width: 100%; padding: 12px; margin: 20px 0; border-radius: 60px; border: 1px solid #cbdde9; text-align: center; font-size: 1rem; }
        .password-modal button { background: #1f5e7e; border: none; padding: 10px; border-radius: 60px; color: white; font-weight: 600; width: 100%; cursor: pointer; }
        .container { max-width: 1200px; margin: 0 auto; display: none; }
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 12px; margin-bottom: 18px; }
        .kpi-card { background: white; border-radius: 24px; padding: 14px 16px; border: 1px solid #e2edf2; box-shadow: 0 2px 8px rgba(0,0,0,0.02); }
        .kpi-title { font-size: 0.7rem; font-weight: 600; color: #5b7a99; margin-bottom: 6px; }
        .kpi-number { font-size: 1.4rem; font-weight: 800; color: #1f4e6e; line-height: 1.2; word-break: break-word; }
        .trend-badge { background: #eef2fa; border-radius: 40px; padding: 3px 10px; font-size: 0.6rem; display: inline-block; margin-top: 6px; }
        .action-toolbar { display: flex; justify-content: flex-end; gap: 12px; margin-bottom: 16px; flex-wrap: wrap; }
        .tool-btn { background: white; border: 1px solid #cbdde9; border-radius: 40px; padding: 6px 16px; font-size: 0.7rem; font-weight: 500; cursor: pointer; color: #2c5778; }
        .tool-btn i { margin-right: 5px; }
        .tool-btn.primary { background: #1f5e7e; border-color: #1f5e7e; color: white; }
        .sync-status { font-size: 0.6rem; background: #eef2fa; border-radius: 20px; padding: 4px 12px; display: inline-flex; align-items: center; gap: 6px; }
        .comparison-card { background: linear-gradient(135deg, #fef9e6 0%, #fff4e0 100%); border-radius: 28px; padding: 18px 20px; margin-bottom: 24px; border-left: 6px solid #e9a23b; }
        .comp-title { font-weight: 800; font-size: 1rem; margin-bottom: 14px; color: #b45f1b; display: flex; align-items: center; gap: 8px; }
        .comp-stats { display: flex; flex-wrap: wrap; gap: 16px; justify-content: space-between; }
        .comp-item { flex: 1; min-width: 140px; background: rgba(255,255,240,0.7); border-radius: 20px; padding: 10px 12px; }
        .comp-label { font-size: 0.65rem; color: #a76f2e; text-transform: uppercase; }
        .comp-value { font-weight: 800; font-size: 1.1rem; color: #2c3e2f; margin-top: 4px; }
        .comp-sub { font-size: 0.7rem; color: #8b6946; margin-top: 4px; }
        .trend-up { color: #c2412c; } .trend-down { color: #2c7a4d; }
        .chart-card { background: white; border-radius: 28px; padding: 16px 12px 12px; margin-bottom: 24px; border: 1px solid #e2edf2; }
        .section-header { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; margin-bottom: 16px; gap: 10px; }
        .year-selector { display: flex; gap: 8px; background: #f8fafd; padding: 4px 10px; border-radius: 60px; }
        .year-btn { background: transparent; border: none; padding: 4px 14px; border-radius: 40px; font-weight: 600; font-size: 0.8rem; cursor: pointer; color: #4f6f8f; }
        .year-btn.active { background: #1f5e7e; color: white; }
        .three-charts-row { display: flex; flex-wrap: wrap; gap: 16px; margin-bottom: 8px; }
        .chart-item { flex: 1; min-width: 260px; height: 300px; }
        @media (max-width: 700px) { .chart-item { min-width: 100%; height: 280px; } }
        .footer-note { text-align: right; font-size: 0.6rem; color: #6f8faa; margin-top: 12px; }
        .tables-row { display: flex; flex-wrap: wrap; gap: 16px; margin-top: 8px; }
        .table-card { flex: 1; background: white; border-radius: 28px; border: 1px solid #e2edf2; overflow: hidden; display: flex; flex-direction: column; }
        .table-header { background: #f9fbfe; padding: 12px 18px; font-weight: 700; font-size: 1rem; border-bottom: 1px solid #e2edf2; }
        .table-content { padding: 14px 16px; flex: 1; }
        .quick-add-bar { display: flex; gap: 10px; flex-wrap: wrap; align-items: center; margin-bottom: 16px; background: #f9fbfe; padding: 10px 12px; border-radius: 24px; }
        .person-btn { background: #eef2fa; border: none; padding: 4px 16px; border-radius: 40px; font-weight: 600; font-size: 0.75rem; cursor: pointer; color: #1f5e7e; }
        .person-btn.active { background: #1f5e7e; color: white; }
        .quick-field { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
        .quick-field select, .quick-field input { padding: 5px 10px; border-radius: 40px; border: 1px solid #cbdde9; background: white; font-size: 0.75rem; }
        .btn-primary { background: #2c7a4d; padding: 5px 14px; color: white; border: none; border-radius: 40px; font-weight: 500; font-size: 0.75rem; cursor: pointer; }
        .data-table { overflow-x: auto; max-height: 420px; overflow-y: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 0.7rem; }
        th, td { padding: 8px 6px; text-align: left; border-bottom: 1px solid #e9f0f3; }
        th { background: #f8fafd; font-weight: 600; color: #2c5778; position: sticky; top: 0; }
        .action-icon { cursor: pointer; margin: 0 2px; background: #ecf3f8; padding: 3px 8px; border-radius: 30px; font-size: 0.65rem; display: inline-block; }
        .toast-msg { position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%); background: #1f2e3a; color: white; padding: 6px 16px; border-radius: 50px; font-size: 0.7rem; z-index: 1100; opacity: 0; transition: 0.2s; pointer-events: none; }
    </style>
</head>
<body>
<div id="passwordOverlay" class="password-overlay">
    <div class="password-modal">
        <i class="fas fa-lock"></i>
        <h3>访问验证</h3>
        <input type="password" id="passwordInput" placeholder="输入密码" autocomplete="off">
        <button id="unlockBtn">进入</button>
        <div id="pwdError" style="color:#c2412c; margin-top:12px; font-size:0.75rem;"></div>
    </div>
</div>

<div class="container" id="mainApp">
    <div class="action-toolbar">
        <div class="sync-status" id="syncStatus"><i class="fas fa-cloud-upload-alt"></i> 连接云端中...</div>
        <button id="manualSaveBtn" class="tool-btn primary"><i class="fas fa-save"></i> 强制同步</button>
        <button id="exportDataBtn" class="tool-btn"><i class="fas fa-download"></i> 导出备份</button>
        <button id="importDataBtn" class="tool-btn"><i class="fas fa-upload"></i> 导入恢复</button>
        <input type="file" id="importFileInput" accept="application/json" style="display: none;" />
    </div>

    <div class="kpi-grid">
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-coins"></i> 最新总欠款</div><div class="kpi-number" id="latestTotalDebt">¥0</div><div class="trend-badge" id="debtDateInfo">—</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均欠款</div><div class="kpi-number" id="avgMonthlyDebt">¥0</div><div class="trend-badge">当年月均</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均消费</div><div class="kpi-number" id="avgMonthlyConsume">¥0</div><div class="trend-badge">工资12.5k+欠款↑+现金↓</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-piggy-bank"></i> 现有存款</div><div class="kpi-number" id="totalDeposit">¥0</div><div class="trend-badge">余额宝+纸币</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-calendar-alt"></i> 最新统计</div><div class="kpi-number" id="latestDateDisplay">—</div><div class="trend-badge">欠款/现金</div></div>
    </div>

    <div class="comparison-card" id="comparisonBoard">
        <div class="comp-title"><i class="fas fa-chart-line"></i> 📊 近期对比洞察</div>
        <div id="compContent">加载中...</div>
    </div>

    <div class="chart-card">
        <div class="section-header">
            <h2 style="font-size:1.1rem;"><i class="fas fa-chart-bar"></i> 月度趋势</h2>
            <div class="year-selector" id="trendYearSelector">
                <button data-year="2026" class="year-btn active">2026</button>
                <button data-year="2025" class="year-btn">2025</button>
                <button data-year="2024" class="year-btn">2024</button>
            </div>
        </div>
        <div class="three-charts-row">
            <div class="chart-item" id="debtBarChart"></div>
            <div class="chart-item" id="cashBarChart"></div>
            <div class="chart-item" id="consumeBarChart"></div>
        </div>
        <div class="footer-note">※ 消费 = 12500 + (上月现金-本月现金) + (本月欠款-上月欠款)</div>
    </div>

    <div class="tables-row">
        <div class="table-card">
            <div class="table-header"><i class="fas fa-list-ul"></i> 欠款明细 · 可编辑</div>
            <div class="table-content">
                <div class="quick-add-bar">
                    <div style="font-weight:600;">➕ 快捷新增：</div>
                    <button id="quickDebtLiangBtn" class="person-btn active">梁</button>
                    <button id="quickDebtWangBtn" class="person-btn">王</button>
                    <div class="quick-field">
                        <select id="debtCategorySelect"><option value="花呗">花呗</option><option value="信用卡">信用卡</option><option value="其他欠款">其他欠款</option></select>
                        <input type="number" id="debtAmountInput" placeholder="金额" step="0.01" value="0" style="width:100px;">
                        <input type="date" id="debtDateInput" value="2026-06-01" style="width:120px;">
                        <button id="addDebtQuickBtn" class="btn-primary"><i class="fas fa-save"></i> 新增</button>
                    </div>
                </div>
                <div class="data-table"><table><thead><tr><th>日期</th><th>欠款人</th><th>类别</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="debtTbody"></tbody></table></div>
            </div>
        </div>
        <div class="table-card">
            <div class="table-header"><i class="fas fa-money-bill-wave"></i> 现金流水 · 可编辑</div>
            <div class="table-content">
                <div class="quick-add-bar">
                    <div style="font-weight:600;">➕ 快捷新增：</div>
                    <button id="quickCashLiangBtn" class="person-btn active">梁</button>
                    <button id="quickCashWangBtn" class="person-btn">王</button>
                    <div class="quick-field">
                        <select id="cashCategorySelect"><option value="余额宝">余额宝</option><option value="纸币">纸币</option><option value="微信">微信</option><option value="支付宝">支付宝</option><option value="银行卡">银行卡</option></select>
                        <input type="number" id="cashAmountInput" placeholder="金额" step="0.01" value="0" style="width:100px;">
                        <input type="date" id="cashDateInput" value="2026-06-01" style="width:120px;">
                        <button id="addCashQuickBtn" class="btn-primary"><i class="fas fa-save"></i> 新增</button>
                    </div>
                </div>
                <div class="data-table"><table><thead><tr><th>日期</th><th>欠款人</th><th>账户</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="cashTbody"></tbody></table></div>
            </div>
        </div>
    </div>
</div>
<div id="toastMsg" class="toast-msg"></div>

<script>
    // ================== 🔥 您的 Firebase 配置（已填入正确的 databaseURL） ==================
    const firebaseConfig = {
        apiKey: "AIzaSyBEfzdtiHPzUOV241Iylw01ZU2WklJTvZc",
        authDomain: "caiwu-dbbca.firebaseapp.com",
        databaseURL: "https://caiwu-dbbca-default-rtdb.asia-southeast1.firebasedatabase.app",
        projectId: "caiwu-dbbca",
        storageBucket: "caiwu-dbbca.firebasestorage.app",
        messagingSenderId: "456445434374",
        appId: "1:456445434374:web:2ae5a1bdaaf4ad03818187",
        measurementId: "G-KPVBE7F58C"
    };
    // ============================================================================

    // 初始化 Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    const debtsRef = db.ref('debts');
    const cashRef = db.ref('cashRecords');
    const metaRef = db.ref('meta');

    // ---------- 完整数据集（2024-2026 全部数据） ----------
    const FULL_DEBTS_DATA = [
        { date: "2024-07-08", debtor: "梁", category: "花呗", amount: 5625.39 }, { date: "2024-07-08", debtor: "梁", category: "白条", amount: 1749.75 }, { date: "2024-07-08", debtor: "梁", category: "信用卡", amount: 9439.38 }, { date: "2024-07-08", debtor: "王", category: "花呗", amount: 7144.69 },
        { date: "2024-07-23", debtor: "梁", category: "花呗", amount: 5136.31 }, { date: "2024-07-23", debtor: "梁", category: "白条", amount: 1166.5 }, { date: "2024-07-23", debtor: "梁", category: "信用卡", amount: 10396.58 }, { date: "2024-07-23", debtor: "王", category: "花呗", amount: 7035.49 },
        { date: "2024-07-30", debtor: "梁", category: "花呗", amount: 5118.34 }, { date: "2024-07-30", debtor: "梁", category: "白条", amount: 1166.5 }, { date: "2024-07-30", debtor: "梁", category: "信用卡", amount: 11992.4 }, { date: "2024-07-30", debtor: "王", category: "花呗", amount: 7692 },
        { date: "2024-08-06", debtor: "梁", category: "花呗", amount: 5170.14 }, { date: "2024-08-06", debtor: "梁", category: "白条", amount: 1166.5 }, { date: "2024-08-06", debtor: "梁", category: "信用卡", amount: 11992.4 }, { date: "2024-08-06", debtor: "王", category: "花呗", amount: 8538 },
        { date: "2024-08-20", debtor: "梁", category: "花呗", amount: 4193.91 }, { date: "2024-08-20", debtor: "梁", category: "白条", amount: 583.25 }, { date: "2024-08-20", debtor: "梁", category: "信用卡", amount: 11819.38 }, { date: "2024-08-20", debtor: "王", category: "花呗", amount: 7882 }, { date: "2024-08-20", debtor: "王", category: "信用卡", amount: 494 },
        { date: "2024-08-26", debtor: "梁", category: "花呗", amount: 4206.45 }, { date: "2024-08-26", debtor: "梁", category: "白条", amount: 583.25 }, { date: "2024-08-26", debtor: "梁", category: "信用卡", amount: 12134.86 }, { date: "2024-08-26", debtor: "王", category: "花呗", amount: 9827.7 }, { date: "2024-08-26", debtor: "王", category: "信用卡", amount: 694 },
        { date: "2024-09-04", debtor: "梁", category: "花呗", amount: 4225.26 }, { date: "2024-09-04", debtor: "梁", category: "白条", amount: 583.25 }, { date: "2024-09-04", debtor: "梁", category: "信用卡", amount: 12134.86 }, { date: "2024-09-04", debtor: "王", category: "花呗", amount: 10866.95 }, { date: "2024-09-04", debtor: "王", category: "信用卡", amount: 694.4 },
        { date: "2024-09-13", debtor: "梁", category: "花呗", amount: 4244.07 }, { date: "2024-09-13", debtor: "梁", category: "白条", amount: 583.25 }, { date: "2024-09-13", debtor: "梁", category: "信用卡", amount: 12134.86 }, { date: "2024-09-13", debtor: "王", category: "花呗", amount: 12781 }, { date: "2024-09-13", debtor: "王", category: "信用卡", amount: 694.4 },
        { date: "2024-09-18", debtor: "梁", category: "花呗", amount: 3954.16 }, { date: "2024-09-18", debtor: "梁", category: "信用卡", amount: 11167.34 }, { date: "2024-09-18", debtor: "王", category: "花呗", amount: 14851.25 },
        { date: "2024-09-27", debtor: "梁", category: "花呗", amount: 4155.32 }, { date: "2024-09-27", debtor: "梁", category: "信用卡", amount: 12135.99 }, { date: "2024-09-27", debtor: "王", category: "花呗", amount: 15344 }, { date: "2024-09-27", debtor: "王", category: "信用卡", amount: 2554 },
        { date: "2024-09-30", debtor: "梁", category: "花呗", amount: 0 }, { date: "2024-09-30", debtor: "梁", category: "信用卡", amount: 0 }, { date: "2024-09-30", debtor: "王", category: "花呗", amount: 15398.6 },
        { date: "2024-10-11", debtor: "梁", category: "花呗", amount: 932.82 }, { date: "2024-10-11", debtor: "梁", category: "信用卡", amount: 313.68 }, { date: "2024-10-11", debtor: "王", category: "花呗", amount: 15644.41 }, { date: "2024-10-11", debtor: "王", category: "信用卡", amount: 466.62 },
        { date: "2024-10-21", debtor: "梁", category: "花呗", amount: 0 }, { date: "2024-10-21", debtor: "梁", category: "信用卡", amount: 0 }, { date: "2024-10-21", debtor: "王", category: "花呗", amount: 14688.05 }, { date: "2024-10-21", debtor: "王", category: "信用卡", amount: 3645.44 },
        { date: "2024-12-02", debtor: "梁", category: "花呗", amount: 1437.89 }, { date: "2024-12-02", debtor: "梁", category: "信用卡", amount: 11839.95 }, { date: "2024-12-02", debtor: "王", category: "花呗", amount: 15447.52 }, { date: "2024-12-02", debtor: "王", category: "信用卡", amount: 1274.07 },
        { date: "2024-12-23", debtor: "梁", category: "花呗", amount: 723.89 }, { date: "2024-12-23", debtor: "梁", category: "信用卡", amount: 11499.95 }, { date: "2024-12-23", debtor: "王", category: "花呗", amount: 14738.17 }, { date: "2024-12-23", debtor: "王", category: "信用卡", amount: 124.6 },
        { date: "2025-01-08", debtor: "梁", category: "花呗", amount: 3133.75 }, { date: "2025-01-08", debtor: "梁", category: "招商信用卡", amount: 13070.24 }, { date: "2025-01-08", debtor: "王", category: "花呗", amount: 15688.25 }, { date: "2025-01-08", debtor: "王", category: "信用卡", amount: 13756.89 },
        { date: "2025-02-06", debtor: "梁", category: "花呗", amount: 2843.18 }, { date: "2025-02-06", debtor: "梁", category: "招商信用卡", amount: 17838.56 }, { date: "2025-02-06", debtor: "王", category: "花呗", amount: 15075.5 }, { date: "2025-02-06", debtor: "王", category: "信用卡", amount: 13778.58 },
        { date: "2025-02-17", debtor: "梁", category: "花呗", amount: 0 }, { date: "2025-02-17", debtor: "梁", category: "招商信用卡", amount: 18608.05 }, { date: "2025-02-17", debtor: "王", category: "花呗", amount: 13166.36 }, { date: "2025-02-17", debtor: "王", category: "信用卡", amount: 12764.77 },
        { date: "2025-03-04", debtor: "梁", category: "花呗", amount: 23.8 }, { date: "2025-03-04", debtor: "梁", category: "招商信用卡", amount: 12910.39 }, { date: "2025-03-04", debtor: "王", category: "花呗", amount: 13564.92 }, { date: "2025-03-04", debtor: "王", category: "信用卡", amount: 10884.88 },
        { date: "2025-03-18", debtor: "梁", category: "花呗", amount: 0 }, { date: "2025-03-18", debtor: "梁", category: "招商信用卡", amount: 11642.35 }, { date: "2025-03-18", debtor: "王", category: "花呗", amount: 14569.5 }, { date: "2025-03-18", debtor: "王", category: "信用卡", amount: 11511.35 },
        { date: "2025-04-05", debtor: "梁", category: "花呗", amount: 175.8 }, { date: "2025-04-05", debtor: "梁", category: "招商信用卡", amount: 13991.04 }, { date: "2025-04-05", debtor: "王", category: "花呗", amount: 14569.5 }, { date: "2025-04-05", debtor: "王", category: "信用卡", amount: 11511.35 },
        { date: "2025-04-15", debtor: "梁", category: "花呗", amount: 213.3 }, { date: "2025-04-15", debtor: "梁", category: "招商信用卡", amount: 14965.4 }, { date: "2025-04-15", debtor: "王", category: "花呗", amount: 13512.21 }, { date: "2025-04-15", debtor: "王", category: "信用卡", amount: 10272.08 },
        { date: "2025-05-09", debtor: "梁", category: "花呗", amount: 724.25 }, { date: "2025-05-09", debtor: "梁", category: "招商信用卡", amount: 14456.01 }, { date: "2025-05-09", debtor: "王", category: "花呗", amount: 14996 }, { date: "2025-05-09", debtor: "王", category: "信用卡", amount: 10440.18 },
        { date: "2025-05-16", debtor: "梁", category: "花呗", amount: 0 }, { date: "2025-05-16", debtor: "梁", category: "招商信用卡", amount: 14978.64 }, { date: "2025-05-16", debtor: "王", category: "花呗", amount: 12383.83 }, { date: "2025-05-16", debtor: "王", category: "信用卡", amount: 9322.31 },
        { date: "2025-06-03", debtor: "梁", category: "花呗", amount: 1792.49 }, { date: "2025-06-03", debtor: "梁", category: "招商信用卡", amount: 22036.79 }, { date: "2025-06-03", debtor: "王", category: "花呗", amount: 13119.7 }, { date: "2025-06-03", debtor: "王", category: "信用卡", amount: 9417.31 },
        { date: "2025-06-10", debtor: "梁", category: "花呗", amount: 1851.29 }, { date: "2025-06-10", debtor: "梁", category: "招商信用卡", amount: 23899.3 }, { date: "2025-06-10", debtor: "王", category: "花呗", amount: 13669.39 }, { date: "2025-06-10", debtor: "王", category: "信用卡", amount: 9477.52 },
        { date: "2025-06-24", debtor: "梁", category: "花呗", amount: 0 }, { date: "2025-06-24", debtor: "梁", category: "招商信用卡", amount: 23316.59 }, { date: "2025-06-24", debtor: "王", category: "花呗", amount: 10262.51 }, { date: "2025-06-24", debtor: "王", category: "信用卡", amount: 12057.14 },
        { date: "2025-07-07", debtor: "梁", category: "花呗", amount: 340.57 }, { date: "2025-07-07", debtor: "梁", category: "招商信用卡", amount: 24780.4 }, { date: "2025-07-07", debtor: "王", category: "花呗", amount: 11403.24 }, { date: "2025-07-07", debtor: "王", category: "信用卡", amount: 13291.79 },
        { date: "2025-07-15", debtor: "梁", category: "花呗", amount: 4528.76 }, { date: "2025-07-15", debtor: "梁", category: "招商信用卡", amount: 25030.14 }, { date: "2025-07-15", debtor: "王", category: "花呗", amount: 8676.74 }, { date: "2025-07-15", debtor: "王", category: "信用卡", amount: 8732.79 },
        { date: "2025-08-06", debtor: "梁", category: "花呗", amount: 4761.2 }, { date: "2025-08-06", debtor: "梁", category: "招商信用卡", amount: 23656.01 }, { date: "2025-08-06", debtor: "王", category: "花呗", amount: 11566.63 }, { date: "2025-08-06", debtor: "王", category: "信用卡", amount: 9134.59 },
        { date: "2025-08-25", debtor: "梁", category: "花呗", amount: 14302.61 }, { date: "2025-08-25", debtor: "梁", category: "招商信用卡", amount: 21639.49 }, { date: "2025-08-25", debtor: "王", category: "花呗", amount: 10913.4 }, { date: "2025-08-25", debtor: "王", category: "信用卡", amount: 4142.06 },
        { date: "2025-09-09", debtor: "梁", category: "花呗", amount: 17551.35 }, { date: "2025-09-09", debtor: "梁", category: "招商信用卡", amount: 22295.19 }, { date: "2025-09-09", debtor: "王", category: "花呗", amount: 12359.34 }, { date: "2025-09-09", debtor: "王", category: "信用卡", amount: 3106.54 },
        { date: "2025-10-09", debtor: "梁", category: "花呗", amount: 12351.33 }, { date: "2025-10-09", debtor: "梁", category: "招商信用卡", amount: 16869.17 }, { date: "2025-10-09", debtor: "王", category: "花呗", amount: 3536.62 }, { date: "2025-10-09", debtor: "王", category: "信用卡", amount: 4795.49 },
        { date: "2025-11-27", debtor: "梁", category: "花呗", amount: 10188.46 }, { date: "2025-11-27", debtor: "梁", category: "招商信用卡", amount: 17508.56 }, { date: "2025-11-27", debtor: "王", category: "花呗", amount: 4778.97 }, { date: "2025-11-27", debtor: "王", category: "信用卡", amount: 2412.56 },
        { date: "2025-12-02", debtor: "梁", category: "花呗", amount: 10785.31 }, { date: "2025-12-02", debtor: "梁", category: "招商信用卡", amount: 17882.03 }, { date: "2025-12-02", debtor: "王", category: "花呗", amount: 5292 }, { date: "2025-12-02", debtor: "王", category: "信用卡", amount: 1186 },
        { date: "2026-01-08", debtor: "梁", category: "花呗", amount: 9604 }, { date: "2026-01-08", debtor: "梁", category: "招商信用卡", amount: 19311.45 }, { date: "2026-01-08", debtor: "王", category: "花呗", amount: 17575.03 }, { date: "2026-01-08", debtor: "王", category: "信用卡", amount: 2401.08 },
        { date: "2026-02-11", debtor: "梁", category: "花呗", amount: 12048.87 }, { date: "2026-02-11", debtor: "梁", category: "招商信用卡", amount: 19987.44 }, { date: "2026-02-11", debtor: "王", category: "花呗", amount: 16756.32 }, { date: "2026-02-11", debtor: "王", category: "信用卡", amount: 7839.58 },
        { date: "2026-02-25", debtor: "梁", category: "花呗", amount: 523.26 }, { date: "2026-02-25", debtor: "王", category: "花呗", amount: 12347.48 }, { date: "2026-03-14", debtor: "梁", category: "花呗", amount: 523.26 }, { date: "2026-03-14", debtor: "王", category: "花呗", amount: 11841.83 },
        { date: "2026-04-14", debtor: "梁", category: "花呗", amount: 1217.82 }, { date: "2026-04-14", debtor: "王", category: "花呗", amount: 14913.66 },
        { date: "2026-05-28", debtor: "梁", category: "招商信用卡（账期到10月，每月500）", amount: 6276.66 }, { date: "2026-05-28", debtor: "王", category: "花呗", amount: 10437.32 }, { date: "2026-05-28", debtor: "王", category: "信用卡", amount: 1647.87 },
        { date: "2026-06-01", debtor: "梁", category: "花呗", amount: 0 }, { date: "2026-06-01", debtor: "梁", category: "招商信用卡（账期到10月，每月500）", amount: 7185.72 }, { date: "2026-06-01", debtor: "王", category: "花呗", amount: 10658.84 }, { date: "2026-06-01", debtor: "王", category: "信用卡", amount: 2252.05 }
    ];
    const FULL_CASH_DATA = [
        { date: "2026-06-01", debtor: "梁", account: "微信", amount: 39 }, { date: "2026-06-01", debtor: "梁", account: "银行卡", amount: 2.67 },
        { date: "2026-06-01", debtor: "王", account: "支付宝", amount: 79 }, { date: "2026-06-01", debtor: "王", account: "余额宝", amount: 11899.41 }, { date: "2026-06-01", debtor: "王", account: "纸币", amount: 1350 },
        { date: "2026-05-28", debtor: "梁", account: "微信", amount: 14.93 }, { date: "2026-05-28", debtor: "梁", account: "银行卡", amount: 2.67 },
        { date: "2026-05-28", debtor: "王", account: "支付宝", amount: 79 }, { date: "2026-05-28", debtor: "王", account: "余额宝", amount: 11922.42 }, { date: "2026-05-28", debtor: "王", account: "纸币", amount: 1350 },
        { date: "2026-05-21", debtor: "梁", account: "微信", amount: 156.83 }, { date: "2026-05-21", debtor: "梁", account: "银行卡", amount: 214.77 },
        { date: "2026-05-21", debtor: "王", account: "支付宝", amount: 79 }, { date: "2026-05-21", debtor: "王", account: "余额宝", amount: 12122.42 }, { date: "2026-05-21", debtor: "王", account: "纸币", amount: 1350 },
        { date: "2026-04-27", debtor: "梁", account: "微信", amount: 71.44 }, { date: "2026-04-27", debtor: "梁", account: "银行卡", amount: 355.69 },
        { date: "2026-04-27", debtor: "王", account: "支付宝", amount: 229.76 }, { date: "2026-04-27", debtor: "王", account: "余额宝", amount: 12117.71 }, { date: "2026-04-27", debtor: "王", account: "纸币", amount: 1850 },
        { date: "2026-04-17", debtor: "梁", account: "微信", amount: 862 }, { date: "2026-04-17", debtor: "梁", account: "银行卡", amount: 1025 },
        { date: "2026-04-17", debtor: "王", account: "支付宝", amount: 452.71 }, { date: "2026-04-17", debtor: "王", account: "余额宝", amount: 12111.14 }, { date: "2026-04-17", debtor: "王", account: "纸币", amount: 1850 }
    ];

    let debts = [], cashRecords = [];
    let nextDebtId = 50000, nextCashId = 60000;
    let currentYear = "2026";
    let debtChart, cashChart, consumeChart;
    let initialSyncDone = false;
    
    function toast(msg) { let t = document.getElementById('toastMsg'); t.innerText = msg; t.style.opacity = '1'; setTimeout(()=> t.style.opacity = '0', 1800); }
    function formatMoneyInt(v) { return Math.round(v).toLocaleString('en-US'); }
    function getTotalDebtByDate(d) { return debts.filter(x=>x.date===d).reduce((s,x)=>s+x.amount,0); }
    function getTotalCashByDate(d) { return cashRecords.filter(x=>x.date===d).reduce((s,x)=>s+x.amount,0); }
    function getDepositByDate(d) { return cashRecords.filter(x=>x.date===d && (x.account==="余额宝"||x.account==="纸币")).reduce((s,x)=>s+x.amount,0); }
    
    function getYearlyMonthlyData(year, type) {
        const src = type === 'debt' ? debts : cashRecords;
        const dates = [...new Set(src.filter(i=>i.date.startsWith(year)).map(i=>i.date))].sort();
        const monthMap = new Map();
        for(let date of dates){
            let month = parseInt(date.slice(5,7),10);
            let val = type==='debt' ? getTotalDebtByDate(date) : getTotalCashByDate(date);
            if(!monthMap.has(month) || date > monthMap.get(month).lastDate) monthMap.set(month, {value: val, lastDate: date});
        }
        let months = Array.from(monthMap.keys()).sort((a,b)=>a-b);
        return { labels: months.map(m=>`${m}月`), values: months.map(m=>monthMap.get(m).value), rawMonths: months };
    }
    
    function getMonthlyConsumption(year){
        const debt = getYearlyMonthlyData(year,'debt');
        const cash = getYearlyMonthlyData(year,'cash');
        if(debt.labels.length===0) return {labels:[],values:[]};
        let allMonthsSet = new Set([...debt.rawMonths, ...cash.rawMonths]);
        let sortedMonths = Array.from(allMonthsSet).sort((a,b)=>a-b);
        let consMap = new Map();
        for(let i=1;i<sortedMonths.length;i++){
            let cur = sortedMonths[i], prev = sortedMonths[i-1];
            let debtCur = debt.values[debt.rawMonths.indexOf(cur)] ?? 0;
            let debtPrev = debt.values[debt.rawMonths.indexOf(prev)] ?? 0;
            let cashCur = cash.values[cash.rawMonths.indexOf(cur)] ?? 0;
            let cashPrev = cash.values[cash.rawMonths.indexOf(prev)] ?? 0;
            let debtInc = Math.max(0, debtCur - debtPrev);
            let cashDec = Math.max(0, cashPrev - cashCur);
            consMap.set(cur, 12500 + debtInc + cashDec);
        }
        let monthsOrder = Array.from(consMap.keys()).sort((a,b)=>a-b);
        return { labels: monthsOrder.map(m=>`${m}月`), values: monthsOrder.map(m=>consMap.get(m)) };
    }
    
    function renderAllCharts(){
        const debtData = getYearlyMonthlyData(currentYear,'debt');
        const cashData = getYearlyMonthlyData(currentYear,'cash');
        const consumeData = getMonthlyConsumption(currentYear);
        if(debtChart) debtChart.dispose();
        debtChart = echarts.init(document.getElementById('debtBarChart'));
        debtChart.setOption({ tooltip:{trigger:'axis',valueFormatter:v=>'¥'+Math.round(v).toLocaleString()}, grid:{containLabel:true}, xAxis:{type:'category',data:debtData.labels}, yAxis:{type:'value',name:'欠款(¥)'}, series:[{type:'bar',data:debtData.values,itemStyle:{color:'#e07a5f',borderRadius:[6,6,0,0],label:{show:true,position:'top',formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        if(cashChart) cashChart.dispose();
        cashChart = echarts.init(document.getElementById('cashBarChart'));
        cashChart.setOption({ tooltip:{trigger:'axis'}, xAxis:{type:'category',data:cashData.labels}, yAxis:{type:'value',name:'现金(¥)'}, series:[{type:'bar',data:cashData.values,itemStyle:{color:'#3b82b6',borderRadius:[6,6,0,0],label:{show:true,formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        if(consumeChart) consumeChart.dispose();
        consumeChart = echarts.init(document.getElementById('consumeBarChart'));
        consumeChart.setOption({ tooltip:{trigger:'axis'}, xAxis:{type:'category',data:consumeData.labels}, yAxis:{type:'value',name:'消费(¥)'}, series:[{type:'bar',data:consumeData.values,itemStyle:{color:'#2c7a4d',borderRadius:[6,6,0,0],label:{show:true,formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        let avgDebt = debtData.values.length ? debtData.values.reduce((a,b)=>a+b,0)/debtData.values.length : 0;
        let avgConsume = consumeData.values.length ? consumeData.values.reduce((a,b)=>a+b,0)/consumeData.values.length : 0;
        document.getElementById('avgMonthlyDebt').innerHTML = `¥${formatMoneyInt(avgDebt)}`;
        document.getElementById('avgMonthlyConsume').innerHTML = `¥${formatMoneyInt(avgConsume)}`;
    }
    
    function formatShortDate(dateStr) { if(!dateStr) return ''; let p=dateStr.split('-'); return `${parseInt(p[1],10)}月${parseInt(p[2],10)}日`; }
    function updateComparisonBoard(){
        let commonDates = [...new Set([...debts.map(d=>d.date), ...cashRecords.map(c=>c.date)])].filter(date => debts.some(d=>d.date===date) && cashRecords.some(c=>c.date===date)).sort().reverse();
        if(commonDates.length < 2){ document.getElementById('compContent').innerHTML = '<div style="padding:12px;text-align:center">📭 暂无两个完整的对比记录</div>'; return; }
        let latestDate = commonDates[0], prevDate = commonDates[1];
        let latestDebt = getTotalDebtByDate(latestDate), prevDebt = getTotalDebtByDate(prevDate);
        let latestDeposit = getDepositByDate(latestDate), prevDeposit = getDepositByDate(prevDate);
        let days = Math.round((new Date(latestDate) - new Date(prevDate)) / (1000*3600*24));
        let debtDiff = latestDebt - prevDebt, debtSymbol = debtDiff >= 0 ? '▲' : '▼', debtAbs = Math.abs(Math.round(debtDiff));
        let depositDiff = latestDeposit - prevDeposit, depositSymbol = depositDiff >= 0 ? '▲' : '▼', depositAbs = Math.abs(Math.round(depositDiff));
        let debtIncrease = Math.max(0, latestDebt - prevDebt);
        let depositDecrease = Math.max(0, prevDeposit - latestDeposit);
        let consumption = debtIncrease + depositDecrease;
        let avgDaily = days ? consumption / days : 0;
        document.getElementById('compContent').innerHTML = `<div class="comp-stats"><div class="comp-item"><div class="comp-label">📅 相隔天数</div><div class="comp-value">${days} 天</div><div class="comp-sub">${formatShortDate(prevDate)} → ${formatShortDate(latestDate)}</div></div><div class="comp-item"><div class="comp-label">💰 欠款变化</div><div class="comp-value ${debtDiff>=0?'trend-up':'trend-down'}">${debtSymbol} ${debtAbs.toLocaleString()}</div><div class="comp-sub">${Math.round(prevDebt).toLocaleString()} → ${Math.round(latestDebt).toLocaleString()}</div></div><div class="comp-item"><div class="comp-label">🏦 存款变化</div><div class="comp-value ${depositDiff>=0?'trend-down':'trend-up'}">${depositSymbol} ${depositAbs.toLocaleString()}</div><div class="comp-sub">${Math.round(prevDeposit).toLocaleString()} → ${Math.round(latestDeposit).toLocaleString()}</div></div><div class="comp-item"><div class="comp-label">🍜 期间总消费</div><div class="comp-value">¥${Math.round(consumption).toLocaleString()}</div><div class="comp-sub">日均 ¥${Math.round(avgDaily).toLocaleString()}</div></div></div>`;
    }
    
    function updateKPI(){
        let allDates = [...new Set(debts.map(d=>d.date))].sort(); let latest = allDates[allDates.length-1];
        if(latest){ document.getElementById('latestTotalDebt').innerHTML = `¥${formatMoneyInt(getTotalDebtByDate(latest))}`; document.getElementById('totalDeposit').innerHTML = `¥${formatMoneyInt(getDepositByDate(latest))}`; document.getElementById('latestDateDisplay').innerHTML = formatShortDate(latest); document.getElementById('debtDateInfo').innerHTML = `截至 ${latest}`; }
        renderAllCharts(); updateComparisonBoard();
    }
    
    function renderDebtTable(){ let tbody = document.getElementById('debtTbody'); tbody.innerHTML = ''; debts.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(d=>{ tbody.innerHTML += `<tr><td>${d.date}</td><td>${d.debtor}</td><td>${d.category}</td><td>¥${Math.round(d.amount).toLocaleString()}</td><td><span class="action-icon edit-debt" data-id="${d.id}">✏️</span> <span class="action-icon del-debt" data-id="${d.id}">🗑️</span></td>`; }); updateKPI(); }
    function renderCashTable(){ let tbody = document.getElementById('cashTbody'); tbody.innerHTML = ''; cashRecords.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(c=>{ tbody.innerHTML += `<tr><td>${c.date}</td><td>${c.debtor}</td><td>${c.account}</td><td>¥${Math.round(c.amount).toLocaleString()}</td><td><span class="action-icon edit-cash" data-id="${c.id}">✏️</span> <span class="action-icon del-cash" data-id="${c.id}">🗑️</span></td>`; }); updateKPI(); }
    
    function pushToCloud(){
        debtsRef.set(debts).catch(e=>console.error);
        cashRef.set(cashRecords).catch(e=>console.error);
        metaRef.set({nextDebtId, nextCashId}).catch(e=>console.error);
        document.getElementById('syncStatus').innerHTML = '<i class="fas fa-check-circle"></i> 已同步';
        setTimeout(()=>{ if(initialSyncDone) document.getElementById('syncStatus').innerHTML = '<i class="fas fa-cloud-upload-alt"></i> 实时同步中'; },2000);
    }
    
    function pullFromCloud(){
        debtsRef.once('value').then(snap=>{ if(snap.exists()) debts = snap.val(); else { initLocalData(); pushToCloud(); } renderDebtTable(); });
        cashRef.once('value').then(snap=>{ if(snap.exists()) cashRecords = snap.val(); else { initLocalData(); pushToCloud(); } renderCashTable(); });
        metaRef.once('value').then(snap=>{ if(snap.exists()){ let m=snap.val(); nextDebtId=m.nextDebtId; nextCashId=m.nextCashId; } });
        initialSyncDone = true;
        document.getElementById('syncStatus').innerHTML = '<i class="fas fa-cloud-upload-alt"></i> 实时同步已开启';
    }
    
    function initLocalData(){
        debts = FULL_DEBTS_DATA.map((d,i)=>({id:nextDebtId+i, ...d, amount:Number(d.amount)}));
        cashRecords = FULL_CASH_DATA.map((c,i)=>({id:nextCashId+i, ...c, amount:Number(c.amount)}));
        nextDebtId += debts.length;
        nextCashId += cashRecords.length;
    }
    
    function listenRealtime(){
        debtsRef.on('value', snap=>{ if(snap.exists()){ debts = snap.val(); renderDebtTable(); } });
        cashRef.on('value', snap=>{ if(snap.exists()){ cashRecords = snap.val(); renderCashTable(); } });
        metaRef.on('value', snap=>{ if(snap.exists()){ let m=snap.val(); nextDebtId=m.nextDebtId; nextCashId=m.nextCashId; } });
    }
    
    function addDebt(r){ let newId = nextDebtId++; let newDebt = {id:newId, ...r}; debts.push(newDebt); renderDebtTable(); pushToCloud(); }
    function updateDebt(id,amt){ let idx=debts.findIndex(d=>d.id===id); if(idx!==-1){ debts[idx].amount=amt; renderDebtTable(); pushToCloud(); } }
    function deleteDebt(id){ debts=debts.filter(d=>d.id!==id); renderDebtTable(); pushToCloud(); }
    function addCash(r){ let newId = nextCashId++; let newCash = {id:newId, ...r}; cashRecords.push(newCash); renderCashTable(); pushToCloud(); }
    function updateCash(id,amt){ let idx=cashRecords.findIndex(c=>c.id===id); if(idx!==-1){ cashRecords[idx].amount=amt; renderCashTable(); pushToCloud(); } }
    function deleteCash(id){ cashRecords=cashRecords.filter(c=>c.id!==id); renderCashTable(); pushToCloud(); }
    
    function exportBackup(){ let dataStr = JSON.stringify({debts,cashRecords,nextDebtId,nextCashId}); let blob = new Blob([dataStr],{type:'application/json'}); let url=URL.createObjectURL(blob); let a=document.createElement('a'); a.href=url; a.download=`finance_backup_${new Date().toISOString().slice(0,19)}.json`; a.click(); URL.revokeObjectURL(url); toast('导出完成'); }
    function importBackup(file){ let reader=new FileReader(); reader.onload=ev=>{ try{ let d=JSON.parse(ev.target.result); if(d.debts && d.cashRecords){ debts=d.debts; cashRecords=d.cashRecords; nextDebtId=d.nextDebtId; nextCashId=d.nextCashId; renderDebtTable(); renderCashTable(); pushToCloud(); toast('导入并已同步云端'); } }catch(e){ toast('文件无效'); } }; reader.readAsText(file); }
    
    document.getElementById('manualSaveBtn').onclick = ()=>{ pushToCloud(); toast('已强制同步'); };
    document.getElementById('exportDataBtn').onclick = exportBackup;
    document.getElementById('importDataBtn').onclick = ()=> document.getElementById('importFileInput').click();
    document.getElementById('importFileInput').onchange = e=>{ if(e.target.files[0]) importBackup(e.target.files[0]); e.target.value=''; };
    
    let quickDebtPerson="梁", quickCashPerson="梁";
    document.getElementById('quickDebtLiangBtn').onclick=()=>{ quickDebtPerson="梁"; document.getElementById('quickDebtLiangBtn').classList.add('active'); document.getElementById('quickDebtWangBtn').classList.remove('active'); };
    document.getElementById('quickDebtWangBtn').onclick=()=>{ quickDebtPerson="王"; document.getElementById('quickDebtWangBtn').classList.add('active'); document.getElementById('quickDebtLiangBtn').classList.remove('active'); };
    document.getElementById('addDebtQuickBtn').onclick=()=>{ let cat=document.getElementById('debtCategorySelect').value, amt=parseFloat(document.getElementById('debtAmountInput').value), dt=document.getElementById('debtDateInput').value; if(!dt||isNaN(amt)){ alert("完整填写"); return; } addDebt({date:dt, debtor:quickDebtPerson, category:cat, amount:amt}); document.getElementById('debtAmountInput').value=0; };
    document.getElementById('quickCashLiangBtn').onclick=()=>{ quickCashPerson="梁"; document.getElementById('quickCashLiangBtn').classList.add('active'); document.getElementById('quickCashWangBtn').classList.remove('active'); };
    document.getElementById('quickCashWangBtn').onclick=()=>{ quickCashPerson="王"; document.getElementById('quickCashWangBtn').classList.add('active'); document.getElementById('quickCashLiangBtn').classList.remove('active'); };
    document.getElementById('addCashQuickBtn').onclick=()=>{ let acc=document.getElementById('cashCategorySelect').value, amt=parseFloat(document.getElementById('cashAmountInput').value), dt=document.getElementById('cashDateInput').value; if(!dt||isNaN(amt)){ alert("完整填写"); return; } addCash({date:dt, debtor:quickCashPerson, account:acc, amount:amt}); document.getElementById('cashAmountInput').value=0; };
    
    document.getElementById('debtTbody').addEventListener('click',(e)=>{ if(e.target.classList.contains('edit-debt')){ let id=parseInt(e.target.dataset.id); let d=debts.find(i=>i.id===id); let na=prompt('修改金额',d.amount); if(na) updateDebt(id,parseFloat(na)); } if(e.target.classList.contains('del-debt')) if(confirm('删除?')) deleteDebt(parseInt(e.target.dataset.id)); });
    document.getElementById('cashTbody').addEventListener('click',(e)=>{ if(e.target.classList.contains('edit-cash')){ let id=parseInt(e.target.dataset.id); let c=cashRecords.find(i=>i.id===id); let na=prompt('修改金额',c.amount); if(na) updateCash(id,parseFloat(na)); } if(e.target.classList.contains('del-cash')) if(confirm('删除?')) deleteCash(parseInt(e.target.dataset.id)); });
    
    document.querySelectorAll('.year-btn').forEach(btn=>{ btn.addEventListener('click',()=>{ document.querySelectorAll('.year-btn').forEach(b=>b.classList.remove('active')); btn.classList.add('active'); currentYear=btn.dataset.year; renderAllCharts(); }); });
    
    const overlay = document.getElementById('passwordOverlay'), unlockBtn = document.getElementById('unlockBtn'), pwdInput = document.getElementById('passwordInput'), mainApp = document.getElementById('mainApp');
    function unlock(){ if(pwdInput.value === "864456"){ overlay.style.display='none'; mainApp.style.display='block'; pullFromCloud(); listenRealtime(); setTimeout(()=>{ if(debtChart) debtChart.resize(); if(cashChart) cashChart.resize(); if(consumeChart) consumeChart.resize(); },200); } else { document.getElementById('pwdError').innerText='密码错误'; pwdInput.value=''; } }
    unlockBtn.onclick = unlock; pwdInput.addEventListener('keypress', e=>{ if(e.key === 'Enter') unlock(); });
    window.addEventListener('resize', ()=>{ setTimeout(()=>{ if(debtChart) debtChart.resize(); if(cashChart) cashChart.resize(); if(consumeChart) consumeChart.resize(); },100); });
</script>
</body>
</html>
