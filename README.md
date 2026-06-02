<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, viewport-fit=cover">
    <title>财务看板</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif; }
        body { background: #f0f4f9; padding: 16px 12px; color: #1a2c3e; }
        .password-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.7); backdrop-filter: blur(6px); display: flex; align-items: center; justify-content: center; z-index: 2000; }
        .password-modal { background: white; border-radius: 40px; padding: 32px 24px; width: 300px; text-align: center; }
        .password-modal i { font-size: 48px; color: #1f5e7e; margin-bottom: 16px; }
        .password-modal input { width: 100%; padding: 12px; margin: 20px 0; border-radius: 60px; border: 1px solid #cbdde9; text-align: center; font-size: 1rem; }
        .password-modal button { background: #1f5e7e; border: none; padding: 10px; border-radius: 60px; color: white; font-weight: 600; width: 100%; cursor: pointer; }
        .container { max-width: 1200px; margin: 0 auto; display: none; }
        /* KPI卡片 */
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 12px; margin-bottom: 18px; }
        .kpi-card { background: white; border-radius: 24px; padding: 14px 16px; border: 1px solid #e2edf2; }
        .kpi-title { font-size: 0.7rem; font-weight: 600; color: #5b7a99; margin-bottom: 6px; }
        .kpi-number { font-size: 1.4rem; font-weight: 800; color: #1f4e6e; line-height: 1.2; }
        .trend-badge { background: #eef2fa; border-radius: 40px; padding: 3px 10px; font-size: 0.6rem; display: inline-block; margin-top: 6px; }
        /* 工具栏 */
        .action-toolbar { display: flex; justify-content: space-between; gap: 12px; margin-bottom: 16px; flex-wrap: wrap; align-items: center; }
        .tool-group { display: flex; gap: 8px; flex-wrap: wrap; }
        .tool-btn { background: white; border: 1px solid #cbdde9; border-radius: 40px; padding: 6px 16px; font-size: 0.7rem; font-weight: 500; cursor: pointer; color: #2c5778; }
        .tool-btn.primary { background: #1f5e7e; border-color: #1f5e7e; color: white; }
        .sync-status { font-size: 0.6rem; background: #eef2fa; border-radius: 20px; padding: 4px 12px; display: inline-flex; align-items: center; gap: 6px; }
        /* 快速录入面板 */
        .batch-panel { background: white; border-radius: 28px; padding: 16px 20px; margin-bottom: 24px; border: 1px solid #e2edf2; }
        .person-switch { display: flex; gap: 12px; margin-bottom: 16px; }
        .person-switch button { background: #eef2fa; border: none; padding: 6px 20px; border-radius: 40px; font-weight: 600; cursor: pointer; }
        .person-switch button.active { background: #1f5e7e; color: white; }
        .batch-form { display: flex; flex-wrap: wrap; gap: 24px; }
        .batch-col { flex: 1; min-width: 240px; }
        .batch-col h4 { font-size: 0.8rem; margin-bottom: 8px; color: #2c5778; border-left: 3px solid #1f5e7e; padding-left: 8px; }
        .input-row { display: flex; align-items: center; gap: 8px; margin-bottom: 8px; justify-content: space-between; }
        .input-row label { font-size: 0.7rem; width: 70px; font-weight: 500; }
        .input-row input { flex: 1; padding: 5px 8px; border-radius: 40px; border: 1px solid #cbdde9; font-size: 0.7rem; text-align: right; }
        .batch-date { display: flex; align-items: center; gap: 12px; margin-top: 12px; padding-top: 12px; border-top: 1px dashed #e2edf2; }
        .batch-date label { font-size: 0.7rem; font-weight: 500; }
        .batch-date input { padding: 5px 10px; border-radius: 40px; border: 1px solid #cbdde9; }
        .save-batch-btn { background: #2c7a4d; color: white; border: none; border-radius: 40px; padding: 6px 20px; font-weight: 600; cursor: pointer; }
        /* 图表 & 表格 */
        .chart-card { background: white; border-radius: 28px; padding: 16px 12px 12px; margin-bottom: 24px; border: 1px solid #e2edf2; }
        .section-header { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; margin-bottom: 16px; gap: 10px; }
        .year-selector { display: flex; gap: 8px; background: #f8fafd; padding: 4px 10px; border-radius: 60px; }
        .year-btn { background: transparent; border: none; padding: 4px 14px; border-radius: 40px; font-weight: 600; font-size: 0.8rem; cursor: pointer; color: #4f6f8f; }
        .year-btn.active { background: #1f5e7e; color: white; }
        .three-charts-row { display: flex; flex-wrap: wrap; gap: 16px; }
        .chart-item { flex: 1; min-width: 260px; height: 300px; }
        @media (max-width:700px){ .chart-item { min-width: 100%; height: 280px; } }
        .footer-note { text-align: right; font-size: 0.6rem; color: #6f8faa; margin-top: 12px; }
        .tables-row { display: flex; flex-wrap: wrap; gap: 16px; margin-top: 8px; }
        .table-card { flex: 1; background: white; border-radius: 28px; border: 1px solid #e2edf2; overflow: hidden; display: flex; flex-direction: column; }
        .table-header { background: #f9fbfe; padding: 12px 18px; font-weight: 700; font-size: 1rem; border-bottom: 1px solid #e2edf2; }
        .table-content { padding: 14px 16px; flex: 1; }
        .data-table { overflow-x: auto; max-height: 420px; overflow-y: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 0.7rem; }
        th, td { padding: 8px 6px; text-align: left; border-bottom: 1px solid #e9f0f3; }
        th { background: #f8fafd; font-weight: 600; color: #2c5778; position: sticky; top: 0; }
        .action-icon { cursor: pointer; margin: 0 2px; background: #ecf3f8; padding: 3px 8px; border-radius: 30px; font-size: 0.65rem; display: inline-block; }
        .toast-msg { position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%); background: #1f2e3a; color: white; padding: 6px 16px; border-radius: 50px; font-size: 0.7rem; z-index: 1100; opacity: 0; transition: 0.2s; pointer-events: none; }
        .date-history { background: white; border-radius: 40px; padding: 4px 12px; display: inline-flex; align-items: center; gap: 8px; }
        .date-history input { border: none; background: transparent; font-size: 0.7rem; font-weight: 500; width: 110px; }
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
        <div class="tool-group">
            <div class="date-history"><i class="fas fa-calendar-alt"></i><input type="date" id="historyDatePicker" value="2026-06-02"></div>
            <button id="loadHistoryBtn" class="tool-btn"><i class="fas fa-eye"></i> 查看快照</button>
            <button id="resetViewBtn" class="tool-btn"><i class="fas fa-sync-alt"></i> 最新数据</button>
        </div>
        <div class="tool-group">
            <div class="sync-status" id="syncStatus"><i class="fas fa-cloud-upload-alt"></i> 同步中</div>
            <button id="manualSaveBtn" class="tool-btn primary"><i class="fas fa-save"></i> 强制同步</button>
            <button id="exportDataBtn" class="tool-btn"><i class="fas fa-download"></i> 导出</button>
            <button id="importDataBtn" class="tool-btn"><i class="fas fa-upload"></i> 导入</button>
            <input type="file" id="importFileInput" accept="application/json" style="display:none" />
        </div>
    </div>

    <!-- KPI 卡片 -->
    <div class="kpi-grid">
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-coins"></i> 总欠款</div><div class="kpi-number" id="latestTotalDebt">¥0</div><div class="trend-badge" id="debtDateInfo">—</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均欠款</div><div class="kpi-number" id="avgMonthlyDebt">¥0</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均消费</div><div class="kpi-number" id="avgMonthlyConsume">¥0</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-piggy-bank"></i> 存款(余额宝+纸币)</div><div class="kpi-number" id="totalDeposit">¥0</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-calendar-alt"></i> 统计日期</div><div class="kpi-number" id="latestDateDisplay">—</div></div>
    </div>

    <!-- 批量录入面板 -->
    <div class="batch-panel">
        <div class="person-switch" id="personSwitch">
            <button data-person="梁" class="active">👤 梁</button>
            <button data-person="王">👤 王</button>
        </div>
        <div class="batch-form" id="batchForm">
            <!-- 动态渲染欠款和现金输入行 -->
        </div>
        <div class="batch-date">
            <label><i class="fas fa-calendar-day"></i> 记账日期</label>
            <input type="date" id="batchDate" value="2026-06-02">
            <button id="saveBatchBtn" class="save-batch-btn"><i class="fas fa-save"></i> 保存所有记录</button>
        </div>
    </div>

    <!-- 月度趋势 -->
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

    <!-- 表格区 -->
    <div class="tables-row">
        <div class="table-card">
            <div class="table-header"><i class="fas fa-list-ul"></i> 欠款明细 · 可编辑</div>
            <div class="table-content"><div class="data-table"><table><thead><tr><th>日期</th><th>欠款人</th><th>类别</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="debtTbody"></tbody></table></div></div>
        </div>
        <div class="table-card">
            <div class="table-header"><i class="fas fa-money-bill-wave"></i> 现金流水 · 可编辑</div>
            <div class="table-content"><div class="data-table"><table><thead><tr><th>日期</th><th>欠款人</th><th>账户</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="cashTbody"></tbody></table></div></div>
        </div>
    </div>
</div>
<div id="toastMsg" class="toast-msg"></div>

<script>
    // ==================== Firebase 配置（用户已提供） ====================
    const firebaseConfig = {
        apiKey: "AIzaSyBEfzdtiHPzUOV241Iylw01ZU2WklJTvZc",
        authDomain: "caiwu-dbbca.firebaseapp.com",
        databaseURL: "https://caiwu-dbbca-default-rtdb.asia-southeast1.firebasedatabase.app",
        projectId: "caiwu-dbbca",
        storageBucket: "caiwu-dbbca.firebasestorage.app",
        messagingSenderId: "456445434374",
        appId: "1:456445434374:web:2ae5a1bdaaf4ad03818187"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    const debtsRef = db.ref('debts');
    const cashRef = db.ref('cashRecords');
    const metaRef = db.ref('meta');

    // ---------- 完整数据集（与用户导出数据一致） ----------
    const FULL_DEBTS_DATA = [{"amount":5625.39,"category":"花呗","date":"2024-07-08","debtor":"梁","id":50000},{"amount":1749.75,"category":"白条","date":"2024-07-08","debtor":"梁","id":50001},{"amount":9439.38,"category":"信用卡","date":"2024-07-08","debtor":"梁","id":50002},{"amount":7144.69,"category":"花呗","date":"2024-07-08","debtor":"王","id":50003},{"amount":5136.31,"category":"花呗","date":"2024-07-23","debtor":"梁","id":50004},{"amount":1166.5,"category":"白条","date":"2024-07-23","debtor":"梁","id":50005},{"amount":10396.58,"category":"信用卡","date":"2024-07-23","debtor":"梁","id":50006},{"amount":7035.49,"category":"花呗","date":"2024-07-23","debtor":"王","id":50007},{"amount":5118.34,"category":"花呗","date":"2024-07-30","debtor":"梁","id":50008},{"amount":1166.5,"category":"白条","date":"2024-07-30","debtor":"梁","id":50009},{"amount":11992.4,"category":"信用卡","date":"2024-07-30","debtor":"梁","id":50010},{"amount":7692,"category":"花呗","date":"2024-07-30","debtor":"王","id":50011},{"amount":5170.14,"category":"花呗","date":"2024-08-06","debtor":"梁","id":50012},{"amount":1166.5,"category":"白条","date":"2024-08-06","debtor":"梁","id":50013},{"amount":11992.4,"category":"信用卡","date":"2024-08-06","debtor":"梁","id":50014},{"amount":8538,"category":"花呗","date":"2024-08-06","debtor":"王","id":50015},{"amount":4193.91,"category":"花呗","date":"2024-08-20","debtor":"梁","id":50016},{"amount":583.25,"category":"白条","date":"2024-08-20","debtor":"梁","id":50017},{"amount":11819.38,"category":"信用卡","date":"2024-08-20","debtor":"梁","id":50018},{"amount":7882,"category":"花呗","date":"2024-08-20","debtor":"王","id":50019},{"amount":494,"category":"信用卡","date":"2024-08-20","debtor":"王","id":50020},{"amount":4206.45,"category":"花呗","date":"2024-08-26","debtor":"梁","id":50021},{"amount":583.25,"category":"白条","date":"2024-08-26","debtor":"梁","id":50022},{"amount":12134.86,"category":"信用卡","date":"2024-08-26","debtor":"梁","id":50023},{"amount":9827.7,"category":"花呗","date":"2024-08-26","debtor":"王","id":50024},{"amount":694,"category":"信用卡","date":"2024-08-26","debtor":"王","id":50025},{"amount":4225.26,"category":"花呗","date":"2024-09-04","debtor":"梁","id":50026},{"amount":583.25,"category":"白条","date":"2024-09-04","debtor":"梁","id":50027},{"amount":12134.86,"category":"信用卡","date":"2024-09-04","debtor":"梁","id":50028},{"amount":10866.95,"category":"花呗","date":"2024-09-04","debtor":"王","id":50029},{"amount":694.4,"category":"信用卡","date":"2024-09-04","debtor":"王","id":50030},{"amount":4244.07,"category":"花呗","date":"2024-09-13","debtor":"梁","id":50031},{"amount":583.25,"category":"白条","date":"2024-09-13","debtor":"梁","id":50032},{"amount":12134.86,"category":"信用卡","date":"2024-09-13","debtor":"梁","id":50033},{"amount":12781,"category":"花呗","date":"2024-09-13","debtor":"王","id":50034},{"amount":694.4,"category":"信用卡","date":"2024-09-13","debtor":"王","id":50035},{"amount":3954.16,"category":"花呗","date":"2024-09-18","debtor":"梁","id":50036},{"amount":11167.34,"category":"信用卡","date":"2024-09-18","debtor":"梁","id":50037},{"amount":14851.25,"category":"花呗","date":"2024-09-18","debtor":"王","id":50038},{"amount":4155.32,"category":"花呗","date":"2024-09-27","debtor":"梁","id":50039},{"amount":12135.99,"category":"信用卡","date":"2024-09-27","debtor":"梁","id":50040},{"amount":15344,"category":"花呗","date":"2024-09-27","debtor":"王","id":50041},{"amount":2554,"category":"信用卡","date":"2024-09-27","debtor":"王","id":50042},{"amount":0,"category":"花呗","date":"2024-09-30","debtor":"梁","id":50043},{"amount":0,"category":"信用卡","date":"2024-09-30","debtor":"梁","id":50044},{"amount":15398.6,"category":"花呗","date":"2024-09-30","debtor":"王","id":50045},{"amount":932.82,"category":"花呗","date":"2024-10-11","debtor":"梁","id":50046},{"amount":313.68,"category":"信用卡","date":"2024-10-11","debtor":"梁","id":50047},{"amount":15644.41,"category":"花呗","date":"2024-10-11","debtor":"王","id":50048},{"amount":466.62,"category":"信用卡","date":"2024-10-11","debtor":"王","id":50049},{"amount":0,"category":"花呗","date":"2024-10-21","debtor":"梁","id":50050},{"amount":0,"category":"信用卡","date":"2024-10-21","debtor":"梁","id":50051},{"amount":14688.05,"category":"花呗","date":"2024-10-21","debtor":"王","id":50052},{"amount":3645.44,"category":"信用卡","date":"2024-10-21","debtor":"王","id":50053},{"amount":1437.89,"category":"花呗","date":"2024-12-02","debtor":"梁","id":50054},{"amount":11839.95,"category":"信用卡","date":"2024-12-02","debtor":"梁","id":50055},{"amount":15447.52,"category":"花呗","date":"2024-12-02","debtor":"王","id":50056},{"amount":1274.07,"category":"信用卡","date":"2024-12-02","debtor":"王","id":50057},{"amount":723.89,"category":"花呗","date":"2024-12-23","debtor":"梁","id":50058},{"amount":11499.95,"category":"信用卡","date":"2024-12-23","debtor":"梁","id":50059},{"amount":14738.17,"category":"花呗","date":"2024-12-23","debtor":"王","id":50060},{"amount":124.6,"category":"信用卡","date":"2024-12-23","debtor":"王","id":50061},{"amount":3133.75,"category":"花呗","date":"2025-01-08","debtor":"梁","id":50062},{"amount":13070.24,"category":"招商信用卡","date":"2025-01-08","debtor":"梁","id":50063},{"amount":15688.25,"category":"花呗","date":"2025-01-08","debtor":"王","id":50064},{"amount":13756.89,"category":"信用卡","date":"2025-01-08","debtor":"王","id":50065},{"amount":2843.18,"category":"花呗","date":"2025-02-06","debtor":"梁","id":50066},{"amount":17838.56,"category":"招商信用卡","date":"2025-02-06","debtor":"梁","id":50067},{"amount":15075.5,"category":"花呗","date":"2025-02-06","debtor":"王","id":50068},{"amount":13778.58,"category":"信用卡","date":"2025-02-06","debtor":"王","id":50069},{"amount":0,"category":"花呗","date":"2025-02-17","debtor":"梁","id":50070},{"amount":18608.05,"category":"招商信用卡","date":"2025-02-17","debtor":"梁","id":50071},{"amount":13166.36,"category":"花呗","date":"2025-02-17","debtor":"王","id":50072},{"amount":12764.77,"category":"信用卡","date":"2025-02-17","debtor":"王","id":50073},{"amount":23.8,"category":"花呗","date":"2025-03-04","debtor":"梁","id":50074},{"amount":12910.39,"category":"招商信用卡","date":"2025-03-04","debtor":"梁","id":50075},{"amount":13564.92,"category":"花呗","date":"2025-03-04","debtor":"王","id":50076},{"amount":10884.88,"category":"信用卡","date":"2025-03-04","debtor":"王","id":50077},{"amount":0,"category":"花呗","date":"2025-03-18","debtor":"梁","id":50078},{"amount":11642.35,"category":"招商信用卡","date":"2025-03-18","debtor":"梁","id":50079},{"amount":14569.5,"category":"花呗","date":"2025-03-18","debtor":"王","id":50080},{"amount":11511.35,"category":"信用卡","date":"2025-03-18","debtor":"王","id":50081},{"amount":175.8,"category":"花呗","date":"2025-04-05","debtor":"梁","id":50082},{"amount":13991.04,"category":"招商信用卡","date":"2025-04-05","debtor":"梁","id":50083},{"amount":14569.5,"category":"花呗","date":"2025-04-05","debtor":"王","id":50084},{"amount":11511.35,"category":"信用卡","date":"2025-04-05","debtor":"王","id":50085},{"amount":213.3,"category":"花呗","date":"2025-04-15","debtor":"梁","id":50086},{"amount":14965.4,"category":"招商信用卡","date":"2025-04-15","debtor":"梁","id":50087},{"amount":13512.21,"category":"花呗","date":"2025-04-15","debtor":"王","id":50088},{"amount":10272.08,"category":"信用卡","date":"2025-04-15","debtor":"王","id":50089},{"amount":724.25,"category":"花呗","date":"2025-05-09","debtor":"梁","id":50090},{"amount":14456.01,"category":"招商信用卡","date":"2025-05-09","debtor":"梁","id":50091},{"amount":14996,"category":"花呗","date":"2025-05-09","debtor":"王","id":50092},{"amount":10440.18,"category":"信用卡","date":"2025-05-09","debtor":"王","id":50093},{"amount":0,"category":"花呗","date":"2025-05-16","debtor":"梁","id":50094},{"amount":14978.64,"category":"招商信用卡","date":"2025-05-16","debtor":"梁","id":50095},{"amount":12383.83,"category":"花呗","date":"2025-05-16","debtor":"王","id":50096},{"amount":9322.31,"category":"信用卡","date":"2025-05-16","debtor":"王","id":50097},{"amount":1792.49,"category":"花呗","date":"2025-06-03","debtor":"梁","id":50098},{"amount":22036.79,"category":"招商信用卡","date":"2025-06-03","debtor":"梁","id":50099},{"amount":13119.7,"category":"花呗","date":"2025-06-03","debtor":"王","id":50100},{"amount":9417.31,"category":"信用卡","date":"2025-06-03","debtor":"王","id":50101},{"amount":1851.29,"category":"花呗","date":"2025-06-10","debtor":"梁","id":50102},{"amount":23899.3,"category":"招商信用卡","date":"2025-06-10","debtor":"梁","id":50103},{"amount":13669.39,"category":"花呗","date":"2025-06-10","debtor":"王","id":50104},{"amount":9477.52,"category":"信用卡","date":"2025-06-10","debtor":"王","id":50105},{"amount":0,"category":"花呗","date":"2025-06-24","debtor":"梁","id":50106},{"amount":23316.59,"category":"招商信用卡","date":"2025-06-24","debtor":"梁","id":50107},{"amount":10262.51,"category":"花呗","date":"2025-06-24","debtor":"王","id":50108},{"amount":12057.14,"category":"信用卡","date":"2025-06-24","debtor":"王","id":50109},{"amount":340.57,"category":"花呗","date":"2025-07-07","debtor":"梁","id":50110},{"amount":24780.4,"category":"招商信用卡","date":"2025-07-07","debtor":"梁","id":50111},{"amount":11403.24,"category":"花呗","date":"2025-07-07","debtor":"王","id":50112},{"amount":13291.79,"category":"信用卡","date":"2025-07-07","debtor":"王","id":50113},{"amount":4528.76,"category":"花呗","date":"2025-07-15","debtor":"梁","id":50114},{"amount":25030.14,"category":"招商信用卡","date":"2025-07-15","debtor":"梁","id":50115},{"amount":8676.74,"category":"花呗","date":"2025-07-15","debtor":"王","id":50116},{"amount":8732.79,"category":"信用卡","date":"2025-07-15","debtor":"王","id":50117},{"amount":4761.2,"category":"花呗","date":"2025-08-06","debtor":"梁","id":50118},{"amount":23656.01,"category":"招商信用卡","date":"2025-08-06","debtor":"梁","id":50119},{"amount":11566.63,"category":"花呗","date":"2025-08-06","debtor":"王","id":50120},{"amount":9134.59,"category":"信用卡","date":"2025-08-06","debtor":"王","id":50121},{"amount":14302.61,"category":"花呗","date":"2025-08-25","debtor":"梁","id":50122},{"amount":21639.49,"category":"招商信用卡","date":"2025-08-25","debtor":"梁","id":50123},{"amount":10913.4,"category":"花呗","date":"2025-08-25","debtor":"王","id":50124},{"amount":4142.06,"category":"信用卡","date":"2025-08-25","debtor":"王","id":50125},{"amount":17551.35,"category":"花呗","date":"2025-09-09","debtor":"梁","id":50126},{"amount":22295.19,"category":"招商信用卡","date":"2025-09-09","debtor":"梁","id":50127},{"amount":12359.34,"category":"花呗","date":"2025-09-09","debtor":"王","id":50128},{"amount":3106.54,"category":"信用卡","date":"2025-09-09","debtor":"王","id":50129},{"amount":12351.33,"category":"花呗","date":"2025-10-09","debtor":"梁","id":50130},{"amount":16869.17,"category":"招商信用卡","date":"2025-10-09","debtor":"梁","id":50131},{"amount":3536.62,"category":"花呗","date":"2025-10-09","debtor":"王","id":50132},{"amount":4795.49,"category":"信用卡","date":"2025-10-09","debtor":"王","id":50133},{"amount":10188.46,"category":"花呗","date":"2025-11-27","debtor":"梁","id":50134},{"amount":17508.56,"category":"招商信用卡","date":"2025-11-27","debtor":"梁","id":50135},{"amount":4778.97,"category":"花呗","date":"2025-11-27","debtor":"王","id":50136},{"amount":2412.56,"category":"信用卡","date":"2025-11-27","debtor":"王","id":50137},{"amount":10785.31,"category":"花呗","date":"2025-12-02","debtor":"梁","id":50138},{"amount":17882.03,"category":"招商信用卡","date":"2025-12-02","debtor":"梁","id":50139},{"amount":5292,"category":"花呗","date":"2025-12-02","debtor":"王","id":50140},{"amount":1186,"category":"信用卡","date":"2025-12-02","debtor":"王","id":50141},{"amount":9604,"category":"花呗","date":"2026-01-08","debtor":"梁","id":50142},{"amount":19311.45,"category":"招商信用卡","date":"2026-01-08","debtor":"梁","id":50143},{"amount":17575.03,"category":"花呗","date":"2026-01-08","debtor":"王","id":50144},{"amount":2401.08,"category":"信用卡","date":"2026-01-08","debtor":"王","id":50145},{"amount":12048.87,"category":"花呗","date":"2026-02-11","debtor":"梁","id":50146},{"amount":19987.44,"category":"招商信用卡","date":"2026-02-11","debtor":"梁","id":50147},{"amount":16756.32,"category":"花呗","date":"2026-02-11","debtor":"王","id":50148},{"amount":7839.58,"category":"信用卡","date":"2026-02-11","debtor":"王","id":50149},{"amount":523.26,"category":"花呗","date":"2026-02-25","debtor":"梁","id":50150},{"amount":12347.48,"category":"花呗","date":"2026-02-25","debtor":"王","id":50151},{"amount":523.26,"category":"花呗","date":"2026-03-14","debtor":"梁","id":50152},{"amount":11841.83,"category":"花呗","date":"2026-03-14","debtor":"王","id":50153},{"amount":1217.82,"category":"花呗","date":"2026-04-14","debtor":"梁","id":50154},{"amount":14913.66,"category":"花呗","date":"2026-04-14","debtor":"王","id":50155},{"amount":6276.66,"category":"招商信用卡（账期到10月，每月500）","date":"2026-05-28","debtor":"梁","id":50156},{"amount":10437.32,"category":"花呗","date":"2026-05-28","debtor":"王","id":50157},{"amount":1647.87,"category":"信用卡","date":"2026-05-28","debtor":"王","id":50158},{"amount":0,"category":"花呗","date":"2026-06-01","debtor":"梁","id":50159},{"amount":7185.72,"category":"招商信用卡（账期到10月，每月500）","date":"2026-06-01","debtor":"梁","id":50160},{"amount":10658.84,"category":"花呗","date":"2026-06-01","debtor":"王","id":50161},{"amount":2252.05,"category":"信用卡","date":"2026-06-01","debtor":"王","id":50162},{"amount":7058,"category":"信用卡","date":"2026-06-02","debtor":"梁","id":50163},{"amount":10760.14,"category":"花呗","date":"2026-06-02","debtor":"王","id":50164},{"amount":2258.42,"category":"信用卡","date":"2026-06-02","debtor":"王","id":50165},{"amount":484.93,"category":"其他欠款","date":"2026-06-02","debtor":"王","id":50166}];
    const FULL_CASH_DATA = [{"account":"微信","amount":39,"date":"2026-06-01","debtor":"梁","id":60000},{"account":"银行卡","amount":2.67,"date":"2026-06-01","debtor":"梁","id":60001},{"account":"支付宝","amount":79,"date":"2026-06-01","debtor":"王","id":60002},{"account":"余额宝","amount":11899.41,"date":"2026-06-01","debtor":"王","id":60003},{"account":"纸币","amount":1350,"date":"2026-06-01","debtor":"王","id":60004},{"account":"微信","amount":14.93,"date":"2026-05-28","debtor":"梁","id":60005},{"account":"银行卡","amount":2.67,"date":"2026-05-28","debtor":"梁","id":60006},{"account":"支付宝","amount":79,"date":"2026-05-28","debtor":"王","id":60007},{"account":"余额宝","amount":11922.42,"date":"2026-05-28","debtor":"王","id":60008},{"account":"纸币","amount":1350,"date":"2026-05-28","debtor":"王","id":60009},{"account":"微信","amount":156.83,"date":"2026-05-21","debtor":"梁","id":60010},{"account":"银行卡","amount":214.77,"date":"2026-05-21","debtor":"梁","id":60011},{"account":"支付宝","amount":79,"date":"2026-05-21","debtor":"王","id":60012},{"account":"余额宝","amount":12122.42,"date":"2026-05-21","debtor":"王","id":60013},{"account":"纸币","amount":1350,"date":"2026-05-21","debtor":"王","id":60014},{"account":"微信","amount":71.44,"date":"2026-04-27","debtor":"梁","id":60015},{"account":"银行卡","amount":355.69,"date":"2026-04-27","debtor":"梁","id":60016},{"account":"支付宝","amount":229.76,"date":"2026-04-27","debtor":"王","id":60017},{"account":"余额宝","amount":12117.71,"date":"2026-04-27","debtor":"王","id":60018},{"account":"纸币","amount":1850,"date":"2026-04-27","debtor":"王","id":60019},{"account":"微信","amount":862,"date":"2026-04-17","debtor":"梁","id":60020},{"account":"银行卡","amount":1025,"date":"2026-04-17","debtor":"梁","id":60021},{"account":"支付宝","amount":452.71,"date":"2026-04-17","debtor":"王","id":60022},{"account":"余额宝","amount":12111.14,"date":"2026-04-17","debtor":"王","id":60023},{"account":"纸币","amount":1850,"date":"2026-04-17","debtor":"王","id":60024},{"account":"银行卡","amount":59,"date":"2026-06-02","debtor":"王","id":60026},{"account":"余额宝","amount":11899.7,"date":"2026-06-02","debtor":"王","id":60028},{"account":"纸币","amount":1350,"date":"2026-06-02","debtor":"王","id":60030}];
    
    let debts = [], cashRecords = [];
    let nextDebtId = 50167, nextCashId = 60031;
    let currentYear = "2026";
    let debtChart, cashChart, consumeChart;
    let currentViewDate = null;  // 当前查看的快照日期，null表示最新
    let originalDebts = [], originalCash = []; // 备份原始数据

    function toast(msg) { let t=document.getElementById('toastMsg'); t.innerText=msg; t.style.opacity='1'; setTimeout(()=>t.style.opacity='0',1800); }
    function roundMoney(v) { return Math.round(v); }
    function formatMoney(v) { return roundMoney(v).toLocaleString('en-US'); }
    function getTotalDebtByDate(d) { return debts.filter(x=>x.date===d).reduce((s,x)=>s+roundMoney(x.amount),0); }
    function getTotalCashByDate(d) { return cashRecords.filter(x=>x.date===d).reduce((s,x)=>s+roundMoney(x.amount),0); }
    function getDepositByDate(d) { return cashRecords.filter(x=>x.date===d && (x.account==="余额宝"||x.account==="纸币")).reduce((s,x)=>s+roundMoney(x.amount),0); }

    function getYearlyMonthlyData(year, type) {
        const src = type === 'debt' ? debts : cashRecords;
        const dates = [...new Set(src.filter(i=>i.date.startsWith(year)).map(i=>i.date))].sort();
        const monthMap = new Map();
        for(let date of dates){
            let month = parseInt(date.slice(5,7));
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
        let allMonths = new Set([...debt.rawMonths, ...cash.rawMonths]);
        let sorted = Array.from(allMonths).sort((a,b)=>a-b);
        let consMap = new Map();
        for(let i=1;i<sorted.length;i++){
            let cur=sorted[i], prev=sorted[i-1];
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
        debtChart.setOption({ tooltip:{trigger:'axis',valueFormatter:v=>'¥'+Math.round(v).toLocaleString()}, xAxis:{type:'category',data:debtData.labels}, yAxis:{type:'value',name:'欠款(¥)'}, series:[{type:'bar',data:debtData.values,itemStyle:{color:'#e07a5f',label:{show:true,position:'top',formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        if(cashChart) cashChart.dispose();
        cashChart = echarts.init(document.getElementById('cashBarChart'));
        cashChart.setOption({ tooltip:{trigger:'axis'}, xAxis:{type:'category',data:cashData.labels}, yAxis:{type:'value',name:'现金(¥)'}, series:[{type:'bar',data:cashData.values,itemStyle:{color:'#3b82b6',label:{show:true,formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        if(consumeChart) consumeChart.dispose();
        consumeChart = echarts.init(document.getElementById('consumeBarChart'));
        consumeChart.setOption({ tooltip:{trigger:'axis'}, xAxis:{type:'category',data:consumeData.labels}, yAxis:{type:'value',name:'消费(¥)'}, series:[{type:'bar',data:consumeData.values,itemStyle:{color:'#2c7a4d',label:{show:true,formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        let avgDebt = debtData.values.length ? debtData.values.reduce((a,b)=>a+b,0)/debtData.values.length : 0;
        let avgConsume = consumeData.values.length ? consumeData.values.reduce((a,b)=>a+b,0)/consumeData.values.length : 0;
        document.getElementById('avgMonthlyDebt').innerHTML = `¥${formatMoney(avgDebt)}`;
        document.getElementById('avgMonthlyConsume').innerHTML = `¥${formatMoney(avgConsume)}`;
    }

    function updateKPI(){
        let targetDate = currentViewDate;
        if(!targetDate){
            let allDates = [...new Set(debts.map(d=>d.date))].sort();
            targetDate = allDates[allDates.length-1];
        }
        if(targetDate){
            document.getElementById('latestTotalDebt').innerHTML = `¥${formatMoney(getTotalDebtByDate(targetDate))}`;
            document.getElementById('totalDeposit').innerHTML = `¥${formatMoney(getDepositByDate(targetDate))}`;
            document.getElementById('latestDateDisplay').innerHTML = targetDate;
            document.getElementById('debtDateInfo').innerHTML = `截至 ${targetDate}`;
        }
        renderAllCharts();
        renderBatchForm();  // 刷新预填表单
    }

    function renderDebtTable(){
        let tbody = document.getElementById('debtTbody');
        tbody.innerHTML = '';
        let filtered = currentViewDate ? debts.filter(d=>d.date===currentViewDate) : debts;
        filtered.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(d=>{
            tbody.innerHTML += `<tr><td>${d.date}</td><td>${d.debtor}</td><td>${d.category}</td><td>¥${formatMoney(d.amount)}</td><td><span class="action-icon edit-debt" data-id="${d.id}">✏️</span> <span class="action-icon del-debt" data-id="${d.id}">🗑️</span></td></tr>`;
        });
        updateKPI();
    }
    function renderCashTable(){
        let tbody = document.getElementById('cashTbody');
        tbody.innerHTML = '';
        let filtered = currentViewDate ? cashRecords.filter(c=>c.date===currentViewDate) : cashRecords;
        filtered.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(c=>{
            tbody.innerHTML += `<tr><td>${c.date}</td><td>${c.debtor}</td><td>${c.account}</td><td>¥${formatMoney(c.amount)}</td><td><span class="action-icon edit-cash" data-id="${c.id}">✏️</span> <span class="action-icon del-cash" data-id="${c.id}">🗑️</span></td></tr>`;
        });
        updateKPI();
    }

    // 预填表单：获取指定人员最近一次记录（按日期）的各分类/账户金额
    function getLatestDebtMap(debtor){
        let latest = debts.filter(d=>d.debtor===debtor).sort((a,b)=>b.date.localeCompare(a.date))[0];
        if(!latest) return new Map();
        let map = new Map();
        debts.filter(d=>d.debtor===debtor && d.date===latest.date).forEach(d=>{ map.set(d.category, roundMoney(d.amount)); });
        return map;
    }
    function getLatestCashMap(debtor){
        let latest = cashRecords.filter(c=>c.debtor===debtor).sort((a,b)=>b.date.localeCompare(a.date))[0];
        if(!latest) return new Map();
        let map = new Map();
        cashRecords.filter(c=>c.debtor===debtor && c.date===latest.date).forEach(c=>{ map.set(c.account, roundMoney(c.amount)); });
        return map;
    }

    let currentPerson = "梁";
    function renderBatchForm(){
        const debtMap = getLatestDebtMap(currentPerson);
        const cashMap = getLatestCashMap(currentPerson);
        // 预定义类别和账户
        const debtCategories = currentPerson === "梁" ? ["花呗","信用卡","其他欠款"] : ["花呗","信用卡","其他欠款"];
        const cashAccounts = currentPerson === "梁" ? ["微信","银行卡","余额宝","纸币","支付宝"] : ["微信","银行卡","余额宝","纸币","支付宝"];
        let html = `<div class="batch-col"><h4>📘 欠款</h4>`;
        debtCategories.forEach(cat => {
            let val = debtMap.get(cat) || 0;
            html += `<div class="input-row"><label>${cat}</label><input type="number" step="1" class="debt-input" data-category="${cat}" value="${val}" placeholder="0"></div>`;
        });
        html += `</div><div class="batch-col"><h4>💰 现金</h4>`;
        cashAccounts.forEach(acc => {
            let val = cashMap.get(acc) || 0;
            html += `<div class="input-row"><label>${acc}</label><input type="number" step="1" class="cash-input" data-account="${acc}" value="${val}" placeholder="0"></div>`;
        });
        html += `</div>`;
        document.getElementById('batchForm').innerHTML = html;
        // 设置默认日期：最新日期+1天，若没有则今天
        let latestDate = debts.map(d=>d.date).sort().reverse()[0] || new Date().toISOString().slice(0,10);
        let nextDate = new Date(latestDate);
        nextDate.setDate(nextDate.getDate()+1);
        document.getElementById('batchDate').value = nextDate.toISOString().slice(0,10);
    }

    function saveBatch(){
        let date = document.getElementById('batchDate').value;
        if(!date){ toast("请选择日期"); return; }
        let debtInputs = document.querySelectorAll('.debt-input');
        let cashInputs = document.querySelectorAll('.cash-input');
        let newDebts = [], newCash = [];
        debtInputs.forEach(inp => {
            let amt = parseFloat(inp.value);
            if(!isNaN(amt) && amt !== 0){
                newDebts.push({ date, debtor: currentPerson, category: inp.dataset.category, amount: amt });
            }
        });
        cashInputs.forEach(inp => {
            let amt = parseFloat(inp.value);
            if(!isNaN(amt) && amt !== 0){
                newCash.push({ date, debtor: currentPerson, account: inp.dataset.account, amount: amt });
            }
        });
        if(newDebts.length===0 && newCash.length===0){ toast("没有非零记录可保存"); return; }
        // 批量添加
        newDebts.forEach(d => addDebt(d, false));
        newCash.forEach(c => addCash(c, false));
        pushToCloud();
        toast(`已保存 ${newDebts.length} 条欠款, ${newCash.length} 条现金`);
        renderBatchForm(); // 刷新表单为新保存的值
        renderDebtTable(); renderCashTable();
    }

    // 云端操作
    function pushToCloud(){ debtsRef.set(debts); cashRef.set(cashRecords); metaRef.set({nextDebtId,nextCashId}); document.getElementById('syncStatus').innerHTML='<i class="fas fa-check-circle"></i> 已同步'; setTimeout(()=>{ document.getElementById('syncStatus').innerHTML='<i class="fas fa-cloud-upload-alt"></i> 实时同步中'; },2000); }
    function pullFromCloud(){
        debtsRef.once('value').then(snap=>{ if(snap.exists()) debts = snap.val(); else initLocalData(); renderDebtTable(); });
        cashRef.once('value').then(snap=>{ if(snap.exists()) cashRecords = snap.val(); else initLocalData(); renderCashTable(); });
        metaRef.once('value').then(snap=>{ if(snap.exists()){ let m=snap.val(); nextDebtId=m.nextDebtId; nextCashId=m.nextCashId; } });
        document.getElementById('syncStatus').innerHTML='<i class="fas fa-cloud-upload-alt"></i> 实时同步已开启';
    }
    function listenRealtime(){
        debtsRef.on('value', snap=>{ if(snap.exists()){ debts = snap.val(); if(!currentViewDate) renderDebtTable(); else renderDebtTable(); } });
        cashRef.on('value', snap=>{ if(snap.exists()){ cashRecords = snap.val(); if(!currentViewDate) renderCashTable(); else renderCashTable(); } });
        metaRef.on('value', snap=>{ if(snap.exists()){ let m=snap.val(); nextDebtId=m.nextDebtId; nextCashId=m.nextCashId; } });
    }
    function initLocalData(){
        debts = FULL_DEBTS_DATA.map(d=>({...d, amount:roundMoney(d.amount)}));
        cashRecords = FULL_CASH_DATA.map(c=>({...c, amount:roundMoney(c.amount)}));
        nextDebtId = Math.max(...debts.map(d=>d.id),50000)+1;
        nextCashId = Math.max(...cashRecords.map(c=>c.id),60000)+1;
    }
    function addDebt(r, sync=true){ let newId=nextDebtId++; debts.push({id:newId, ...r, amount:roundMoney(r.amount)}); if(sync) pushToCloud(); }
    function addCash(r, sync=true){ let newId=nextCashId++; cashRecords.push({id:newId, ...r, amount:roundMoney(r.amount)}); if(sync) pushToCloud(); }
    function updateDebt(id,amt){ let idx=debts.findIndex(d=>d.id===id); if(idx!==-1){ debts[idx].amount=roundMoney(amt); renderDebtTable(); pushToCloud(); } }
    function deleteDebt(id){ debts=debts.filter(d=>d.id!==id); renderDebtTable(); pushToCloud(); }
    function updateCash(id,amt){ let idx=cashRecords.findIndex(c=>c.id===id); if(idx!==-1){ cashRecords[idx].amount=roundMoney(amt); renderCashTable(); pushToCloud(); } }
    function deleteCash(id){ cashRecords=cashRecords.filter(c=>c.id!==id); renderCashTable(); pushToCloud(); }

    // 历史快照
    function loadHistorySnapshot(){
        let date = document.getElementById('historyDatePicker').value;
        if(!date) return;
        currentViewDate = date;
        renderDebtTable();
        renderCashTable();
        toast(`查看 ${date} 快照`);
    }
    function resetToLatest(){
        currentViewDate = null;
        renderDebtTable();
        renderCashTable();
        toast("已恢复最新数据");
    }

    // 事件绑定
    document.getElementById('manualSaveBtn').onclick = pushToCloud;
    document.getElementById('exportDataBtn').onclick = ()=>{ let dataStr=JSON.stringify({debts,cashRecords,nextDebtId,nextCashId}); let blob=new Blob([dataStr],{type:'application/json'}); let a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download=`finance_${new Date().toISOString().slice(0,19)}.json`; a.click(); URL.revokeObjectURL(blob); toast('导出完成'); };
    document.getElementById('importDataBtn').onclick = ()=> document.getElementById('importFileInput').click();
    document.getElementById('importFileInput').onchange = e=>{ let file=e.target.files[0]; if(!file) return; let reader=new FileReader(); reader.onload=ev=>{ try{ let d=JSON.parse(ev.target.result); if(d.debts && d.cashRecords){ debts=d.debts; cashRecords=d.cashRecords; nextDebtId=d.nextDebtId; nextCashId=d.nextCashId; renderDebtTable(); renderCashTable(); pushToCloud(); toast('导入成功并同步'); } }catch(e){ toast('文件无效'); } }; reader.readAsText(file); e.target.value=''; };
    document.getElementById('loadHistoryBtn').onclick = loadHistorySnapshot;
    document.getElementById('resetViewBtn').onclick = resetToLatest;
    document.getElementById('saveBatchBtn').onclick = saveBatch;
    document.querySelectorAll('#personSwitch button').forEach(btn=>{
        btn.onclick = ()=>{
            document.querySelectorAll('#personSwitch button').forEach(b=>b.classList.remove('active'));
            btn.classList.add('active');
            currentPerson = btn.dataset.person;
            renderBatchForm();
        };
    });
    document.getElementById('debtTbody').addEventListener('click',(e)=>{
        if(e.target.classList.contains('edit-debt')){ let id=parseInt(e.target.dataset.id); let d=debts.find(i=>i.id===id); let na=prompt('修改金额',d.amount); if(na) updateDebt(id,parseFloat(na)); }
        if(e.target.classList.contains('del-debt')) if(confirm('删除?')) deleteDebt(parseInt(e.target.dataset.id));
    });
    document.getElementById('cashTbody').addEventListener('click',(e)=>{
        if(e.target.classList.contains('edit-cash')){ let id=parseInt(e.target.dataset.id); let c=cashRecords.find(i=>i.id===id); let na=prompt('修改金额',c.amount); if(na) updateCash(id,parseFloat(na)); }
        if(e.target.classList.contains('del-cash')) if(confirm('删除?')) deleteCash(parseInt(e.target.dataset.id));
    });
    document.querySelectorAll('.year-btn').forEach(btn=>{
        btn.addEventListener('click',()=>{
            document.querySelectorAll('.year-btn').forEach(b=>b.classList.remove('active'));
            btn.classList.add('active');
            currentYear=btn.dataset.year;
            renderAllCharts();
        });
    });

    // 密码解锁
    const overlay=document.getElementById('passwordOverlay'), unlockBtn=document.getElementById('unlockBtn'), pwdInput=document.getElementById('passwordInput'), mainApp=document.getElementById('mainApp');
    function unlock(){ if(pwdInput.value==="864456"){ overlay.style.display='none'; mainApp.style.display='block'; pullFromCloud(); listenRealtime(); setTimeout(()=>{ renderAllCharts(); },200); } else { document.getElementById('pwdError').innerText='密码错误'; pwdInput.value=''; } }
    unlockBtn.onclick = unlock;
    pwdInput.addEventListener('keypress', e=>{ if(e.key==='Enter') unlock(); });
    window.addEventListener('resize', ()=>setTimeout(()=>{ if(debtChart) debtChart.resize(); if(cashChart) cashChart.resize(); if(consumeChart) consumeChart.resize(); },100));
</script>
</body>
</html>
