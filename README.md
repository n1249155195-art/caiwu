<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, viewport-fit=cover">
    <title>欠款现金看板</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Inter', system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif; }
        body { background: #f0f4f9; padding: 16px 12px; color: #1a2c3e; }
        .password-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.7); backdrop-filter: blur(6px);
            display: flex; align-items: center; justify-content: center; z-index: 2000;
        }
        .password-modal {
            background: white; border-radius: 40px; padding: 32px 24px; width: 300px; text-align: center;
        }
        .password-modal i { font-size: 48px; color: #1f5e7e; margin-bottom: 16px; }
        .password-modal input { width: 100%; padding: 12px; margin: 20px 0; border-radius: 60px; border: 1px solid #cbdde9; text-align: center; font-size: 1rem; }
        .password-modal button { background: #1f5e7e; border: none; padding: 10px; border-radius: 60px; color: white; font-weight: 600; width: 100%; cursor: pointer; }
        .container { max-width: 1200px; margin: 0 auto; display: none; }
        
        /* 顶部保存按钮 */
        .top-bar {
            display: flex;
            justify-content: flex-end;
            margin-bottom: 12px;
            gap: 8px;
        }
        .save-btn {
            background: #2c7a4d;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 0.8rem;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 4px;
        }
        .save-btn:hover {
            opacity: 0.9;
        }

        /* KPI 卡片 - 更紧凑适应手机 */
        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 12px;
            margin-bottom: 20px;
        }
        .kpi-card {
            background: white; border-radius: 24px; padding: 14px 16px;
            border: 1px solid #e2edf2; box-shadow: 0 2px 8px rgba(0,0,0,0.02);
        }
        .kpi-title { font-size: 0.7rem; font-weight: 600; color: #5b7a99; margin-bottom: 6px; }
        .kpi-number { font-size: 1.4rem; font-weight: 800; color: #1f4e6e; line-height: 1.2; word-break: break-word; }
        .trend-badge { background: #eef2fa; border-radius: 40px; padding: 3px 10px; font-size: 0.6rem; display: inline-block; margin-top: 6px; }

        /* 新增间隔统计看板 */
        .stats-card {
            background: #fff9e6;
            border: 1px solid #ffdca8;
            border-radius: 20px;
            padding: 12px 16px;
            margin-bottom: 20px;
        }
        .stats-title {
            font-size: 0.85rem;
            font-weight: 700;
            color: #b76b00;
            margin-bottom: 8px;
        }
        .stats-grid {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            font-size: 0.75rem;
        }
        .stats-item {
            flex: 1;
            min-width: 120px;
            background: white;
            padding: 6px 10px;
            border-radius: 12px;
            border: 1px solid #ffe8c3;
        }
        .stats-label {
            color: #7d5a00;
        }
        .stats-value {
            font-weight: 700;
            color: #c17500;
        }

        /* 图表卡片 */
        .chart-card {
            background: white; border-radius: 28px; padding: 16px 12px 12px 12px;
            margin-bottom: 24px; border: 1px solid #e2edf2;
        }
        .section-header {
            display: flex; justify-content: space-between; align-items: center;
            flex-wrap: wrap; margin-bottom: 16px; gap: 10px;
        }
        .year-selector {
            display: flex; gap: 8px; background: #f8fafd; padding: 4px 10px; border-radius: 60px;
        }
        .year-btn {
            background: transparent; border: none; padding: 4px 14px; border-radius: 40px;
            font-weight: 600; font-size: 0.8rem; cursor: pointer; color: #4f6f8f;
        }
        .year-btn.active { background: #1f5e7e; color: white; }
        /* 三图一行 - 移动端改为垂直，PC端同行 */
        .three-charts-row {
            display: flex;
            flex-wrap: wrap;
            gap: 16px;
            margin-bottom: 8px;
        }
        .chart-item {
            flex: 1;
            min-width: 260px;
            height: 300px;
        }
        @media (max-width: 700px) {
            .chart-item { min-width: 100%; height: 280px; }
        }
        .footer-note { text-align: right; font-size: 0.6rem; color: #6f8faa; margin-top: 12px; }
        /* 双表格并列 */
        .tables-row {
            display: flex; flex-wrap: wrap; gap: 16px; margin-top: 8px;
        }
        .table-card {
            flex: 1; background: white; border-radius: 28px; border: 1px solid #e2edf2;
            overflow: hidden; display: flex; flex-direction: column;
        }
        .table-header { background: #f9fbfe; padding: 12px 18px; font-weight: 700; font-size: 1rem; border-bottom: 1px solid #e2edf2; }
        .table-content { padding: 14px 16px; flex: 1; }
        .quick-add-bar {
            display: flex; gap: 10px; flex-wrap: wrap; align-items: center;
            margin-bottom: 16px; background: #f9fbfe; padding: 10px 12px; border-radius: 24px;
        }
        .person-btn {
            background: #eef2fa; border: none; padding: 4px 16px; border-radius: 40px;
            font-weight: 600; font-size: 0.75rem; cursor: pointer; color: #1f5e7e;
        }
        .person-btn.active { background: #1f5e7e; color: white; }
        .quick-field { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
        .quick-field select, .quick-field input {
            padding: 5px 10px; border-radius: 40px; border: 1px solid #cbdde9;
            background: white; font-size: 0.75rem;
        }
        .btn-primary { background: #2c7a4d; padding: 5px 14px; color: white; border: none; border-radius: 40px; font-weight: 500; font-size: 0.75rem; cursor: pointer; }
        .data-table { overflow-x: auto; max-height: 420px; overflow-y: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 0.7rem; }
        th, td { padding: 8px 6px; text-align: left; border-bottom: 1px solid #e9f0f3; }
        th { background: #f8fafd; font-weight: 600; color: #2c5778; position: sticky; top: 0; }
        .action-icon { cursor: pointer; margin: 0 2px; background: #ecf3f8; padding: 3px 8px; border-radius: 30px; font-size: 0.65rem; display: inline-block; }
        @media (max-width: 680px) {
            .quick-field { width: 100%; justify-content: space-between; }
            .kpi-number { font-size: 1.2rem; }
        }
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
    <!-- 顶部手动保存按钮 -->
    <div class="top-bar">
        <button class="save-btn" id="manualSaveBtn"><i class="fas fa-save"></i> 保存数据</button>
    </div>

    <!-- KPI 卡片 -->
    <div class="kpi-grid">
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-coins"></i> 最新总欠款</div><div class="kpi-number" id="latestTotalDebt">¥0</div><div class="trend-badge" id="debtDateInfo">—</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均欠款</div><div class="kpi-number" id="avgMonthlyDebt">¥0</div><div class="trend-badge">当年月均</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均消费</div><div class="kpi-number" id="avgMonthlyConsume">¥0</div><div class="trend-badge">工资12.5k+欠款↑+现金↓</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-piggy-bank"></i> 现有存款</div><div class="kpi-number" id="totalDeposit">¥0</div><div class="trend-badge">余额宝+纸币</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-calendar-alt"></i> 最新统计</div><div class="kpi-number" id="latestDateDisplay">—</div><div class="trend-badge">欠款/现金</div></div>
    </div>

    <!-- 新增：最近记录间隔统计看板 -->
    <div class="stats-card" id="intervalStats">
        <div class="stats-title">📊 最近两次记录统计</div>
        <div class="stats-grid">
            <div class="stats-item">
                <span class="stats-label">间隔天数：</span>
                <span class="stats-value" id="intervalDays">-</span>
            </div>
            <div class="stats-item">
                <span class="stats-label">总消费：</span>
                <span class="stats-value" id="totalConsume">-</span>
            </div>
            <div class="stats-item">
                <span class="stats-label">日均消费：</span>
                <span class="stats-value" id="avgDailyConsume">-</span>
            </div>
            <div class="stats-item">
                <span class="stats-label">欠款浮动：</span>
                <span class="stats-value" id="debtFloat">-</span>
            </div>
            <div class="stats-item">
                <span class="stats-label">存款浮动：</span>
                <span class="stats-value" id="cashFloat">-</span>
            </div>
        </div>
    </div>

    <!-- 三图一行 -->
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

    <!-- 双表格：欠款 + 现金 -->
    <div class="tables-row">
        <div class="table-card">
            <div class="table-header"><i class="fas fa-list-ul"></i> 欠款明细 · 可编辑</div>
            <div class="table-content">
                <div class="quick-add-bar">
                    <div style="font-weight:600;">➕ 快捷新增：</div>
                    <button id="quickDebtLiangBtn" class="person-btn active">梁</button>
                    <button id="quickDebtWangBtn" class="person-btn">王</button>
                    <div class="quick-field">
                        <select id="debtCategorySelect">
                            <option value="花呗">花呗</option>
                            <option value="信用卡">信用卡</option>
                            <option value="其他欠款">其他欠款</option>
                        </select>
                        <input type="number" id="debtAmountInput" placeholder="金额" step="0.01" value="0" style="width:100px;">
                        <input type="date" id="debtDateInput" style="width:120px;">
                        <button id="addDebtQuickBtn" class="btn-primary"><i class="fas fa-save"></i> 新增</button>
                    </div>
                </div>
                <div class="data-table">
                    <table><thead><tr><th>日期</th><th>欠款人</th><th>类别</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="debtTbody"></tbody></table>
                </div>
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
                        <select id="cashCategorySelect">
                            <option value="余额宝">余额宝</option>
                            <option value="纸币">纸币</option>
                            <option value="其他现金">其他现金</option>
                        </select>
                        <input type="number" id="cashAmountInput" placeholder="金额" step="0.01" value="0" style="width:100px;">
                        <input type="date" id="cashDateInput" style="width:120px;">
                        <button id="addCashQuickBtn" class="btn-primary"><i class="fas fa-save"></i> 新增</button>
                    </div>
                </div>
                <div class="data-table">
                    <table><thead><tr><th>日期</th><th>欠款人</th><th>账户</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="cashTbody"></tbody></table>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
    // --------------------------------------------------------------
    // 完整欠款数据集 (2024-2026 基于用户历史)
    // --------------------------------------------------------------
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

    // 现金数据 (仅2026年)
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

    // 工具函数
    function formatMoneyInt(v) { return Math.round(v).toLocaleString('en-US'); }
    function formatMoney(v) { return '¥' + v.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,'); }
    function getTotalDebtByDate(d) { return debts.filter(x=>x.date===d).reduce((s,x)=>s+x.amount,0); }
    function getTotalCashByDate(d) { return cashRecords.filter(x=>x.date===d).reduce((s,x)=>s+x.amount,0); }
    function getDepositByDate(d) { return cashRecords.filter(x=>x.date===d && (x.account==="余额宝"||x.account==="纸币")).reduce((s,x)=>s+x.amount,0); }
    
    // 计算两个日期相差天数
    function getDayDiff(date1, date2) {
        const a = new Date(date1);
        const b = new Date(date2);
        const diffTime = Math.abs(b - a);
        return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
    }

    // 获取所有记录日期（去重+排序）
    function getAllRecordDates() {
        const debtDates = debts.map(i => i.date);
        const cashDates = cashRecords.map(i => i.date);
        const allDates = [...new Set([...debtDates, ...cashDates])];
        return allDates.sort((a, b) => new Date(a) - new Date(b));
    }

    // 渲染最近记录统计看板
    function renderIntervalStats() {
        const dates = getAllRecordDates();
        if (dates.length < 2) {
            document.getElementById('intervalDays').textContent = '数据不足';
            document.getElementById('totalConsume').textContent = '-';
            document.getElementById('avgDailyConsume').textContent = '-';
            document.getElementById('debtFloat').textContent = '-';
            document.getElementById('cashFloat').textContent = '-';
            return;
        }

        // 最近两次日期
        const lastDate = dates[dates.length - 1];
        const prevDate = dates[dates.length - 2];
        const days = getDayDiff(lastDate, prevDate);

        // 计算欠款/存款浮动
        const lastDebt = getTotalDebtByDate(lastDate);
        const prevDebt = getTotalDebtByDate(prevDate);
        const debtChange = lastDebt - prevDebt;
        const debtFloatText = debtChange >= 0 ? `↑${formatMoneyInt(debtChange)}` : `↓${formatMoneyInt(Math.abs(debtChange))}`;

        const lastCash = getDepositByDate(lastDate);
        const prevCash = getDepositByDate(prevDate);
        const cashChange = lastCash - prevCash;
        const cashFloatText = cashChange >= 0 ? `↑${formatMoneyInt(cashChange)}` : `↓${formatMoneyInt(Math.abs(cashChange))}`;

        // 计算总消费 & 日均消费
        const debtInc = Math.max(0, lastDebt - prevDebt);
        const cashDec = Math.max(0, prevCash - lastCash);
        const totalConsume = 12500 + debtInc + cashDec;
        const avgDaily = days > 0 ? (totalConsume / days) : 0;

        // 赋值到页面
        document.getElementById('intervalDays').textContent = `${days} 天`;
        document.getElementById('totalConsume').textContent = `¥${formatMoneyInt(totalConsume)}`;
        document.getElementById('avgDailyConsume').textContent = `¥${formatMoneyInt(avgDaily)}`;
        document.getElementById('debtFloat').textContent = debtFloatText;
        document.getElementById('cashFloat').textContent = cashFloatText;
    }

    function getYearlyMonthlyData(year, type) {
        const src = type === 'debt' ? debts : cashRecords;
        const dates = [...new Set(src.filter(i=>i.date.startsWith(year)).map(i=>i.date))].sort();
        const map = new Map();
        for(let date of dates){
            let month = parseInt(date.slice(5,7),10);
            let val = type==='debt' ? getTotalDebtByDate(date) : getTotalCashByDate(date);
            if(!map.has(month) || date > map.get(month).lastDate) map.set(month,{value:val,lastDate:date});
        }
        let months = Array.from(map.keys()).sort((a,b)=>a-b);
        return { labels: months.map(m=>`${m}月`), values: months.map(m=>map.get(m).value), rawMonths: months };
    }

    function getMonthlyConsumption(year){
        const debt = getYearlyMonthlyData(year,'debt');
        const cash = getYearlyMonthlyData(year,'cash');
        if(debt.labels.length===0) return {labels:[],values:[]};
        let allMonths = new Set(debt.rawMonths);
        cash.rawMonths.forEach(m=>allMonths.add(m));
        let sorted = Array.from(allMonths).sort((a,b)=>a-b);
        let consMap = new Map();
        for(let i=1;i<sorted.length;i++){
            let cur = sorted[i], prev = sorted[i-1];
            let debtCur = debt.values[debt.rawMonths.indexOf(cur)] ?? 0;
            let debtPrev = debt.values[debt.rawMonths.indexOf(prev)] ?? 0;
            let cashCur = cash.values[cash.rawMonths.indexOf(cur)] ?? 0;
            let cashPrev = cash.values[cash.rawMonths.indexOf(prev)] ?? 0;
            let debtInc = Math.max(0, debtCur - debtPrev);
            let cashDec = Math.max(0, cashPrev - cashCur);
            consMap.set(cur, 12500 + debtInc + cashDec);
        }
        let monthsOrder = Array.from(consMap.keys()).sort();
        return { labels: monthsOrder.map(m=>`${m}月`), values: monthsOrder.map(m=>consMap.get(m)) };
    }

    function resizeAllCharts() {
        if(debtChart) debtChart.resize();
        if(cashChart) cashChart.resize();
        if(consumeChart) consumeChart.resize();
    }
    window.addEventListener('resize', () => { setTimeout(resizeAllCharts, 100); });

    function renderAllCharts(){
        const debtData = getYearlyMonthlyData(currentYear,'debt');
        const cashData = getYearlyMonthlyData(currentYear,'cash');
        const consumeData = getMonthlyConsumption(currentYear);
        if(debtChart) debtChart.dispose();
        debtChart = echarts.init(document.getElementById('debtBarChart'));
        debtChart.setOption({ tooltip:{trigger:'axis',valueFormatter:v=>'¥'+v.toFixed(2)}, grid:{containLabel:true}, xAxis:{type:'category',data:debtData.labels,name:'月份',axisLabel:{rotate:0}}, yAxis:{type:'value',name:'欠款(¥)',axisLabel:{formatter:v=>'¥'+Math.round(v).toLocaleString()}}, series:[{type:'bar',data:debtData.values,itemStyle:{color:'#e07a5f',borderRadius:[6,6,0,0],label:{show:true,position:'top',formatter:p=>Math.round(p.value).toLocaleString(),fontWeight:'bold'}}}] });
        if(cashChart) cashChart.dispose();
        cashChart = echarts.init(document.getElementById('cashBarChart'));
        cashChart.setOption({ tooltip:{trigger:'axis',valueFormatter:v=>'¥'+v.toFixed(2)}, grid:{containLabel:true}, xAxis:{type:'category',data:cashData.labels,name:'月份'}, yAxis:{type:'value',name:'现金(¥)',axisLabel:{formatter:v=>'¥'+Math.round(v).toLocaleString()}}, series:[{type:'bar',data:cashData.values,itemStyle:{color:'#3b82b6',borderRadius:[6,6,0,0],label:{show:true,position:'top',formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        if(consumeChart) consumeChart.dispose();
        consumeChart = echarts.init(document.getElementById('consumeBarChart'));
        consumeChart.setOption({ tooltip:{trigger:'axis',valueFormatter:v=>'¥'+v.toFixed(2)}, grid:{containLabel:true}, xAxis:{type:'category',data:consumeData.labels,name:'月份'}, yAxis:{type:'value',name:'消费(¥)',axisLabel:{formatter:v=>'¥'+Math.round(v).toLocaleString()}}, series:[{type:'bar',data:consumeData.values,itemStyle:{color:'#2c7a4d',borderRadius:[6,6,0,0],label:{show:true,position:'top',formatter:p=>Math.round(p.value).toLocaleString()}}}] });
        let avgDebt = debtData.values.length ? debtData.values.reduce((a,b)=>a+b,0)/debtData.values.length : 0;
        let avgConsume = consumeData.values.length ? consumeData.values.reduce((a,b)=>a+b,0)/consumeData.values.length : 0;
        document.getElementById('avgMonthlyDebt').innerHTML = `¥${formatMoneyInt(avgDebt)}`;
        document.getElementById('avgMonthlyConsume').innerHTML = `¥${formatMoneyInt(avgConsume)}`;
    }

    function updateKPI(){
        let allDates = [...new Set(debts.map(d=>d.date))].sort();
        let latest = allDates[allDates.length-1];
        if(latest){
            document.getElementById('latestTotalDebt').innerHTML = `¥${formatMoneyInt(getTotalDebtByDate(latest))}`;
            document.getElementById('totalDeposit').innerHTML = `¥${formatMoneyInt(getDepositByDate(latest))}`;
            document.getElementById('latestDateDisplay').innerHTML = latest;
            document.getElementById('debtDateInfo').innerHTML = `截至 ${latest}`;
        }
        // 渲染统计看板
        renderIntervalStats();
        renderAllCharts();
    }

    function renderDebtTable(){
        let tbody = document.getElementById('debtTbody');
        tbody.innerHTML = '';
        debts.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(d=>{
            tbody.innerHTML += `<tr><td>${d.date}</td><td>${d.debtor}</td><td>${d.category}</td><td>¥${d.amount.toFixed(2)}</td><td><span class="action-icon edit-debt" data-id="${d.id}">✏️</span> <span class="action-icon del-debt" data-id="${d.id}">🗑️</span></td></tr>`;
        });
        updateKPI();
    }
    function renderCashTable(){
        let tbody = document.getElementById('cashTbody');
        tbody.innerHTML = '';
        cashRecords.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(c=>{
            tbody.innerHTML += `<tr><td>${c.date}</td><td>${c.debtor}</td><td>${c.account}</td><td>¥${c.amount.toFixed(2)}</td><td><span class="action-icon edit-cash" data-id="${c.id}">✏️</span> <span class="action-icon del-cash" data-id="${c.id}">🗑️</span></td></tr>`;
        });
        updateKPI();
    }

    // 数据初始化 / 本地存储
    function initData(){
        debts = FULL_DEBTS_DATA.map((d,i)=>({id:nextDebtId+i, ...d, amount:Number(d.amount.toFixed(2))}));
        cashRecords = FULL_CASH_DATA.map((c,i)=>({id:nextCashId+i, ...c, amount:Number(c.amount)}));
        nextDebtId += debts.length; nextCashId += cashRecords.length;
        // 初始化后自动保存
        saveLocal();
    }
    // 统一保存函数 - 所有操作都会调用，确保实时保存
    function saveLocal(){ 
        localStorage.setItem('financeFinalV2', JSON.stringify({debts,cashRecords,nextDebtId,nextCashId}));
        console.log('✅ 数据已自动保存到本地');
    }
    // 手动保存
    function manualSave() {
        saveLocal();
        alert('✅ 数据已手动保存成功！');
    }
    function loadLocal(){
        let raw = localStorage.getItem('financeFinalV2');
        if(raw){ 
            try{ 
                let d=JSON.parse(raw); 
                debts=d.debts; 
                cashRecords=d.cashRecords; 
                nextDebtId=d.nextDebtId; 
                nextCashId=d.nextCashId; 
                console.log('✅ 从本地存储加载数据成功');
            }catch(e){ 
                console.error('本地数据损坏，重新初始化', e);
                initData(); 
            } 
        } else {
            console.log('🆕 首次使用，初始化数据');
            initData(); 
        }
        renderDebtTable(); 
        renderCashTable();
    }

    // 数据操作（增删改）后 自动保存
    function addDebt(r){ debts.push({id:nextDebtId++, ...r}); renderDebtTable(); saveLocal(); }
    function updateDebt(id,amt){ let idx=debts.findIndex(d=>d.id===id); if(idx!==-1){ debts[idx].amount=amt; renderDebtTable(); saveLocal(); } }
    function deleteDebt(id){ debts=debts.filter(d=>d.id!==id); renderDebtTable(); saveLocal(); }
    function addCash(r){ cashRecords.push({id:nextCashId++, ...r}); renderCashTable(); saveLocal(); }
    function updateCash(id,amt){ let idx=cashRecords.findIndex(c=>c.id===id); if(idx!==-1){ cashRecords[idx].amount=amt; renderCashTable(); saveLocal(); } }
    function deleteCash(id){ cashRecords=cashRecords.filter(c=>c.id!==id); renderCashTable(); saveLocal(); }

    // 快捷新增
    let quickDebtPerson="梁", quickCashPerson="梁";
    document.getElementById('quickDebtLiangBtn').onclick=()=>{ quickDebtPerson="梁"; document.getElementById('quickDebtLiangBtn').classList.add('active'); document.getElementById('quickDebtWangBtn').classList.remove('active'); };
    document.getElementById('quickDebtWangBtn').onclick=()=>{ quickDebtPerson="王"; document.getElementById('quickDebtWangBtn').classList.add('active'); document.getElementById('quickDebtLiangBtn').classList.remove('active'); };
    document.getElementById('addDebtQuickBtn').onclick=()=>{
        let cat=document.getElementById('debtCategorySelect').value, amt=parseFloat(document.getElementById('debtAmountInput').value), dt=document.getElementById('debtDateInput').value;
        if(!dt||isNaN(amt)){ alert("完整填写"); return; }
        addDebt({date:dt, debtor:quickDebtPerson, category:cat, amount:amt});
        document.getElementById('debtAmountInput').value=0;
    };
    document.getElementById('quickCashLiangBtn').onclick=()=>{ quickCashPerson="梁"; document.getElementById('quickCashLiangBtn').classList.add('active'); document.getElementById('quickCashWangBtn').classList.remove('active'); };
    document.getElementById('quickCashWangBtn').onclick=()=>{ quickCashPerson="王"; document.getElementById('quickCashWangBtn').classList.add('active'); document.getElementById('quickCashLiangBtn').classList.remove('active'); };
    document.getElementById('addCashQuickBtn').onclick=()=>{
        let acc=document.getElementById('cashCategorySelect').value, amt=parseFloat(document.getElementById('cashAmountInput').value), dt=document.getElementById('cashDateInput').value;
        if(!dt||isNaN(amt)){ alert("完整填写"); return; }
        addCash({date:dt, debtor:quickCashPerson, account:acc, amount:amt});
        document.getElementById('cashAmountInput').value=0;
    };

    // 编辑删除事件
    document.getElementById('debtTbody').addEventListener('click',(e)=>{
        if(e.target.classList.contains('edit-debt')){ let id=parseInt(e.target.dataset.id); let d=debts.find(i=>i.id===id); let na=prompt('修改金额',d.amount); if(na) updateDebt(id,parseFloat(na)); }
        if(e.target.classList.contains('del-debt')) if(confirm('删除?')) deleteDebt(parseInt(e.target.dataset.id));
    });
    document.getElementById('cashTbody').addEventListener('click',(e)=>{
        if(e.target.classList.contains('edit-cash')){ let id=parseInt(e.target.dataset.id); let c=cashRecords.find(i=>c.id===id); let na=prompt('修改金额',c.amount); if(na) updateCash(id,parseFloat(na)); }
        if(e.target.classList.contains('del-cash')) if(confirm('删除?')) deleteCash(parseInt(e.target.dataset.id));
    });

    // 年份切换
    document.querySelectorAll('.year-btn').forEach(btn=>{
        btn.addEventListener('click',()=>{
            document.querySelectorAll('.year-btn').forEach(b=>b.classList.remove('active'));
            btn.classList.add('active');
            currentYear=btn.dataset.year;
            renderAllCharts();
        });
    });

    // 密码解锁
    const overlay = document.getElementById('passwordOverlay');
    const unlockBtn = document.getElementById('unlockBtn');
    const pwdInput = document.getElementById('passwordInput');
    const mainApp = document.getElementById('mainApp');
    function unlock(){ if(pwdInput.value === "864456"){ overlay.style.display='none'; mainApp.style.display='block'; loadLocal(); setTimeout(resizeAllCharts, 200); } else { document.getElementById('pwdError').innerText='密码错误'; pwdInput.value=''; } }
    unlockBtn.onclick = unlock;
    pwdInput.addEventListener('keypress', e=>{ if(e.key === 'Enter') unlock(); });

    // 绑定手动保存按钮
    document.getElementById('manualSaveBtn').addEventListener('click', manualSave);

    // 设置默认日期为今天
    document.addEventListener('DOMContentLoaded', function() {
        const today = new Date().toISOString().split('T')[0];
        document.getElementById('debtDateInput').value = today;
        document.getElementById('cashDateInput').value = today;
    });
</script>
</body>
</html>  
