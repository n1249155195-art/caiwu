<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>睿财看板 | 欠款+现金+消费 | 三图一行</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Inter', system-ui, sans-serif; }
        body { background: #f0f4f9; padding: 24px 20px; color: #1a2c3e; }
        .password-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.7); backdrop-filter: blur(6px);
            display: flex; align-items: center; justify-content: center; z-index: 2000;
        }
        .password-modal {
            background: white; border-radius: 40px; padding: 32px 28px; width: 320px; text-align: center;
        }
        .password-modal i { font-size: 48px; color: #1f5e7e; margin-bottom: 16px; }
        .password-modal input { width: 100%; padding: 12px; margin: 20px 0; border-radius: 60px; border: 1px solid #cbdde9; text-align: center; }
        .password-modal button { background: #1f5e7e; border: none; padding: 10px; border-radius: 60px; color: white; font-weight: 600; width: 100%; cursor: pointer; }
        .container { max-width: 1600px; margin: 0 auto; display: none; }
        /* KPI 卡片 */
        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 22px;
            margin-bottom: 28px;
        }
        .kpi-card {
            background: white; border-radius: 28px; padding: 20px 24px;
            border: 1px solid #e2edf2; box-shadow: 0 4px 12px rgba(0,0,0,0.02);
        }
        .kpi-title { font-size: 0.85rem; font-weight: 600; color: #5b7a99; margin-bottom: 10px; }
        .kpi-number { font-size: 1.9rem; font-weight: 800; color: #1f4e6e; line-height: 1.2; word-break: break-word; }
        .trend-badge { background: #eef2fa; border-radius: 40px; padding: 4px 12px; font-size: 0.7rem; display: inline-block; margin-top: 8px; }
        /* 图表卡片 */
        .chart-card {
            background: white; border-radius: 28px; padding: 20px 20px 12px 20px;
            margin-bottom: 32px; border: 1px solid #e2edf2;
        }
        .section-header {
            display: flex; justify-content: space-between; align-items: center;
            flex-wrap: wrap; margin-bottom: 20px; gap: 12px;
        }
        .year-selector {
            display: flex; gap: 12px; background: #f8fafd; padding: 5px 12px; border-radius: 60px;
        }
        .year-btn {
            background: transparent; border: none; padding: 6px 20px; border-radius: 40px;
            font-weight: 600; cursor: pointer; color: #4f6f8f;
        }
        .year-btn.active { background: #1f5e7e; color: white; }
        /* 三图一行 */
        .three-charts-row {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            margin-bottom: 10px;
        }
        .chart-item {
            flex: 1;
            min-width: 260px;
            height: 340px;
        }
        .footer-note { text-align: right; font-size: 0.7rem; color: #6f8faa; margin-top: 16px; }
        /* 双表格并列 */
        .tables-row {
            display: flex; flex-wrap: wrap; gap: 24px; margin-top: 8px;
        }
        .table-card {
            flex: 1; background: white; border-radius: 28px; border: 1px solid #e2edf2;
            overflow: hidden; display: flex; flex-direction: column;
        }
        .table-header { background: #f9fbfe; padding: 16px 24px; font-weight: 700; font-size: 1.2rem; border-bottom: 1px solid #e2edf2; }
        .table-content { padding: 20px 24px; flex: 1; }
        .quick-add-bar {
            display: flex; gap: 12px; flex-wrap: wrap; align-items: center;
            margin-bottom: 20px; background: #f9fbfe; padding: 12px 16px; border-radius: 24px;
        }
        .person-btn {
            background: #eef2fa; border: none; padding: 6px 20px; border-radius: 40px;
            font-weight: 600; cursor: pointer; color: #1f5e7e;
        }
        .person-btn.active { background: #1f5e7e; color: white; }
        .quick-field { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
        .quick-field select, .quick-field input {
            padding: 6px 12px; border-radius: 40px; border: 1px solid #cbdde9;
            background: white; font-size: 0.8rem;
        }
        .btn-primary { background: #2c7a4d; padding: 6px 18px; color: white; border: none; border-radius: 40px; font-weight: 500; cursor: pointer; }
        .data-table { overflow-x: auto; max-height: 500px; overflow-y: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 0.8rem; }
        th, td { padding: 10px 8px; text-align: left; border-bottom: 1px solid #e9f0f3; }
        th { background: #f8fafd; font-weight: 600; color: #2c5778; position: sticky; top: 0; }
        .action-icon { cursor: pointer; margin: 0 4px; background: #ecf3f8; padding: 4px 8px; border-radius: 30px; font-size: 0.7rem; display: inline-block; }
        @media (max-width: 1000px) { .three-charts-row { flex-direction: column; } .tables-row { flex-direction: column; } }
    </style>
</head>
<body>
<div id="passwordOverlay" class="password-overlay">
    <div class="password-modal">
        <i class="fas fa-lock"></i>
        <h3>财务看板 · 安全验证</h3>
        <input type="password" id="passwordInput" placeholder="请输入访问密码" autocomplete="off">
        <button id="unlockBtn">解锁进入</button>
        <div id="pwdError" style="color:#c2412c; margin-top:12px; font-size:0.8rem;"></div>
    </div>
</div>

<div class="container" id="mainApp">
    <!-- KPI 卡片 -->
    <div class="kpi-grid">
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-coins"></i> 最新总欠款</div><div class="kpi-number" id="latestTotalDebt">¥0</div><div class="trend-badge" id="debtDateInfo">—</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均欠款 (当年)</div><div class="kpi-number" id="avgMonthlyDebt">¥0</div><div class="trend-badge">各月欠款平均值</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-chart-line"></i> 月均消费 (当年)</div><div class="kpi-number" id="avgMonthlyConsume">¥0</div><div class="trend-badge">工资12.5k + 欠款增加 + 现金减少</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-piggy-bank"></i> 现有存款 (余额宝+纸币)</div><div class="kpi-number" id="totalDeposit">¥0</div><div class="trend-badge">稳健资产</div></div>
        <div class="kpi-card"><div class="kpi-title"><i class="fas fa-calendar-alt"></i> 最新统计日期</div><div class="kpi-number" id="latestDateDisplay">—</div><div class="trend-badge">欠款/现金同步</div></div>
    </div>

    <!-- 三图一行：欠款柱状图 + 现金柱状图 + 消费柱状图 -->
    <div class="chart-card">
        <div class="section-header">
            <h2><i class="fas fa-chart-bar"></i> 月度趋势 · 柱状分析</h2>
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
        <div class="footer-note">※ 消费 = 12500 + (上月现金-本月现金) + (本月欠款-上月欠款)；无现金数据时仅按欠款增加 + 12500 估算</div>
    </div>

    <!-- 双表格并列 -->
    <div class="tables-row">
        <div class="table-card">
            <div class="table-header"><i class="fas fa-list-ul"></i> 欠款明细表 · 可编辑</div>
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
                        <input type="number" id="debtAmountInput" placeholder="金额(¥)" step="0.01" value="0">
                        <input type="date" id="debtDateInput" value="2026-06-01">
                        <button id="addDebtQuickBtn" class="btn-primary"><i class="fas fa-save"></i> 新增</button>
                    </div>
                </div>
                <div class="data-table">
                    <table><thead><tr><th>统计时间</th><th>欠款人</th><th>类别</th><th>金额(¥)</th><th>操作</th></tr></thead><tbody id="debtTbody"></tbody></table>
                </div>
            </div>
        </div>
        <div class="table-card">
            <div class="table-header"><i class="fas fa-money-bill-wave"></i> 现金流水明细 · 可编辑</div>
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
                        <input type="number" id="cashAmountInput" placeholder="金额(¥)" step="0.01" value="0">
                        <input type="date" id="cashDateInput" value="2026-06-01">
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
    // ======================= 完整欠款数据 (2024-2026) =======================
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

    // 现金数据 (仅2026年有数据，2024/2025 无记录)
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

    function formatMoneyInt(v) { return Math.round(v).toLocaleString('en-US'); }

    function getTotalDebtByDate(date) { return debts.filter(d => d.date === date).reduce((s, d) => s + d.amount, 0); }
    function getTotalCashByDate(date) { return cashRecords.filter(c => c.date === date).reduce((s, c) => s + c.amount, 0); }
    function getDepositByDate(date) { return cashRecords.filter(c => c.date === date && (c.account === "余额宝" || c.account === "纸币")).reduce((s,c) => s + c.amount, 0); }

    // 获取年份中各月份最后一条数据的欠款/现金总额
    function getYearlyMonthlyData(year, type) {
        const source = type === 'debt' ? debts : cashRecords;
        const yearDates = [...new Set(source.filter(i => i.date.startsWith(year)).map(i => i.date))].sort();
        const monthlyMap = new Map();
        for (let date of yearDates) {
            const monthNum = parseInt(date.substring(5,7), 10);
            const value = type === 'debt' ? getTotalDebtByDate(date) : getTotalCashByDate(date);
            if (!monthlyMap.has(monthNum) || date > monthlyMap.get(monthNum).lastDate) {
                monthlyMap.set(monthNum, { value, lastDate: date });
            }
        }
        const sorted = Array.from(monthlyMap.keys()).sort((a,b)=>a-b);
        const labels = sorted.map(m => `${m}月`);
        const values = sorted.map(m => monthlyMap.get(m).value);
        return { labels, values, rawMonths: sorted };
    }

    // 消费计算: 支持无现金年份（仅用欠款增加 + 12500）
    function getMonthlyConsumption(year) {
        const debtMonthly = getYearlyMonthlyData(year, 'debt');
        const cashMonthly = getYearlyMonthlyData(year, 'cash');
        if (debtMonthly.labels.length === 0) return { labels: [], values: [] };
        // 所有月份集合（欠款月份为主，现金如果有则合并）
        let allMonthsSet = new Set(debtMonthly.rawMonths);
        if (cashMonthly.rawMonths.length) cashMonthly.rawMonths.forEach(m => allMonthsSet.add(m));
        const sortedMonths = Array.from(allMonthsSet).sort((a,b)=>a-b);
        const consumptionMap = new Map();
        for (let i = 1; i < sortedMonths.length; i++) {
            const monthCurr = sortedMonths[i];
            const monthPrev = sortedMonths[i-1];
            // 欠款值（必有）
            const debtCurr = debtMonthly.values[debtMonthly.rawMonths.indexOf(monthCurr)] ?? 0;
            const debtPrev = debtMonthly.values[debtMonthly.rawMonths.indexOf(monthPrev)] ?? 0;
            const debtIncrease = Math.max(0, debtCurr - debtPrev);
            // 现金值，若当年无现金数据则 cashCurr = cashPrev = 0，现金减少为0
            let cashCurr = 0, cashPrev = 0;
            if (cashMonthly.rawMonths.length) {
                cashCurr = cashMonthly.values[cashMonthly.rawMonths.indexOf(monthCurr)] ?? 0;
                cashPrev = cashMonthly.values[cashMonthly.rawMonths.indexOf(monthPrev)] ?? 0;
            }
            const cashDecrease = Math.max(0, cashPrev - cashCurr);
            const consume = 12500 + debtIncrease + cashDecrease;
            consumptionMap.set(monthCurr, consume);
        }
        const monthsOrder = Array.from(consumptionMap.keys()).sort();
        const labels = monthsOrder.map(m => `${m}月`);
        const values = monthsOrder.map(m => consumptionMap.get(m));
        return { labels, values };
    }

    function renderAllCharts() {
        // 欠款柱状图
        const debtData = getYearlyMonthlyData(currentYear, 'debt');
        if (debtChart) debtChart.dispose();
        debtChart = echarts.init(document.getElementById('debtBarChart'));
        debtChart.setOption({
            tooltip: { trigger: 'axis', valueFormatter: v => '¥' + v.toFixed(2) },
            xAxis: { type: 'category', data: debtData.labels, name: '月份' },
            yAxis: { type: 'value', name: '欠款总额 (¥)', axisLabel: { formatter: val => '¥' + Math.round(val).toLocaleString() } },
            series: [{ type: 'bar', data: debtData.values, itemStyle: { color: '#e07a5f', borderRadius: [8,8,0,0], label: { show: true, position: 'top', formatter: p => Math.round(p.value).toLocaleString(), fontWeight: 'bold' } } }]
        });
        // 现金柱状图
        const cashData = getYearlyMonthlyData(currentYear, 'cash');
        if (cashChart) cashChart.dispose();
        cashChart = echarts.init(document.getElementById('cashBarChart'));
        cashChart.setOption({
            tooltip: { trigger: 'axis', valueFormatter: v => '¥' + v.toFixed(2) },
            xAxis: { type: 'category', data: cashData.labels, name: '月份' },
            yAxis: { type: 'value', name: '现金总额 (¥)', axisLabel: { formatter: val => '¥' + Math.round(val).toLocaleString() } },
            series: [{ type: 'bar', data: cashData.values, itemStyle: { color: '#3b82b6', borderRadius: [8,8,0,0], label: { show: true, position: 'top', formatter: p => Math.round(p.value).toLocaleString(), fontWeight: 'bold' } } }]
        });
        // 消费柱状图
        const consumeData = getMonthlyConsumption(currentYear);
        if (consumeChart) consumeChart.dispose();
        consumeChart = echarts.init(document.getElementById('consumeBarChart'));
        consumeChart.setOption({
            tooltip: { trigger: 'axis', valueFormatter: v => '¥' + v.toFixed(2) },
            xAxis: { type: 'category', data: consumeData.labels, name: '月份' },
            yAxis: { type: 'value', name: '月均消费 (¥)', axisLabel: { formatter: val => '¥' + Math.round(val).toLocaleString() } },
            series: [{ type: 'bar', data: consumeData.values, itemStyle: { color: '#2c7a4d', borderRadius: [8,8,0,0], label: { show: true, position: 'top', formatter: p => Math.round(p.value).toLocaleString(), fontWeight: 'bold' } } }]
        });
        // 更新KPI: 月均欠款 + 月均消费
        const avgDebt = debtData.values.length ? debtData.values.reduce((a,b)=>a+b,0)/debtData.values.length : 0;
        document.getElementById('avgMonthlyDebt').innerHTML = `¥${formatMoneyInt(avgDebt)}`;
        const avgConsume = consumeData.values.length ? consumeData.values.reduce((a,b)=>a+b,0)/consumeData.values.length : 0;
        document.getElementById('avgMonthlyConsume').innerHTML = `¥${formatMoneyInt(avgConsume)}`;
    }

    function updateKPI() {
        const allDebtDates = [...new Set(debts.map(d=>d.date))].sort();
        const latestDate = allDebtDates[allDebtDates.length-1];
        if (latestDate) {
            const totalDebt = getTotalDebtByDate(latestDate);
            const deposit = getDepositByDate(latestDate);
            document.getElementById('latestTotalDebt').innerHTML = `¥${formatMoneyInt(totalDebt)}`;
            document.getElementById('totalDeposit').innerHTML = `¥${formatMoneyInt(deposit)}`;
            document.getElementById('latestDateDisplay').innerHTML = latestDate;
            document.getElementById('debtDateInfo').innerHTML = `截至 ${latestDate}`;
        }
        renderAllCharts();
    }

    function renderDebtTable() {
        const tbody = document.getElementById('debtTbody');
        tbody.innerHTML = '';
        debts.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(d => {
            tbody.innerHTML += `<tr><td>${d.date}</td><td>${d.debtor}</td><td>${d.category}</td><td>¥${d.amount.toFixed(2)}</td><td><span class="action-icon edit-debt" data-id="${d.id}">✏️编辑</span> <span class="action-icon del-debt" data-id="${d.id}">删除</span></td></tr>`;
        });
        updateKPI();
    }
    function renderCashTable() {
        const tbody = document.getElementById('cashTbody');
        tbody.innerHTML = '';
        cashRecords.slice().sort((a,b)=>b.date.localeCompare(a.date)).forEach(c => {
            tbody.innerHTML += `<tr><td>${c.date}</td><td>${c.debtor}</td><td>${c.account}</td><td>¥${c.amount.toFixed(2)}</td><td><span class="action-icon edit-cash" data-id="${c.id}">✏️编辑</span> <span class="action-icon del-cash" data-id="${c.id}">删除</span></td></tr>`;
        });
        updateKPI();
    }

    function initData() {
        debts = FULL_DEBTS_DATA.map((d,idx) => ({ id: nextDebtId+idx, ...d, amount: Number(d.amount.toFixed(2)) }));
        cashRecords = FULL_CASH_DATA.map((c,idx) => ({ id: nextCashId+idx, ...c, amount: Number(c.amount) }));
        nextDebtId += debts.length;
        nextCashId += cashRecords.length;
    }
    function saveToLocal() { localStorage.setItem('financeFullFinal', JSON.stringify({ debts, cashRecords, nextDebtId, nextCashId })); }
    function loadLocal() {
        const raw = localStorage.getItem('financeFullFinal');
        if (raw) { try { const d=JSON.parse(raw); debts=d.debts; cashRecords=d.cashRecords; nextDebtId=d.nextDebtId; nextCashId=d.nextCashId; } catch(e){ initData(); } }
        else initData();
        renderDebtTable(); renderCashTable();
    }

    function addDebt(rec) { debts.push({ id: nextDebtId++, ...rec }); renderDebtTable(); saveToLocal(); }
    function updateDebt(id, amt) { const idx=debts.findIndex(d=>d.id===id); if(idx!==-1){ debts[idx].amount=amt; renderDebtTable(); saveToLocal(); } }
    function deleteDebt(id) { debts=debts.filter(d=>d.id!==id); renderDebtTable(); saveToLocal(); }
    function addCash(rec) { cashRecords.push({ id: nextCashId++, ...rec }); renderCashTable(); saveToLocal(); }
    function updateCash(id, amt) { const idx=cashRecords.findIndex(c=>c.id===id); if(idx!==-1){ cashRecords[idx].amount=amt; renderCashTable(); saveToLocal(); } }
    function deleteCash(id) { cashRecords=cashRecords.filter(c=>c.id!==id); renderCashTable(); saveToLocal(); }

    // 快捷新增欠款
    let quickDebtPerson = "梁";
    document.getElementById('quickDebtLiangBtn').onclick = () => { quickDebtPerson="梁"; document.getElementById('quickDebtLiangBtn').classList.add('active'); document.getElementById('quickDebtWangBtn').classList.remove('active'); };
    document.getElementById('quickDebtWangBtn').onclick = () => { quickDebtPerson="王"; document.getElementById('quickDebtWangBtn').classList.add('active'); document.getElementById('quickDebtLiangBtn').classList.remove('active'); };
    document.getElementById('addDebtQuickBtn').onclick = () => {
        const category = document.getElementById('debtCategorySelect').value;
        const amount = parseFloat(document.getElementById('debtAmountInput').value);
        const date = document.getElementById('debtDateInput').value;
        if (!date || isNaN(amount)) { alert("完整填写"); return; }
        addDebt({ date, debtor: quickDebtPerson, category, amount });
        document.getElementById('debtAmountInput').value = 0;
    };
    let quickCashPerson = "梁";
    document.getElementById('quickCashLiangBtn').onclick = () => { quickCashPerson="梁"; document.getElementById('quickCashLiangBtn').classList.add('active'); document.getElementById('quickCashWangBtn').classList.remove('active'); };
    document.getElementById('quickCashWangBtn').onclick = () => { quickCashPerson="王"; document.getElementById('quickCashWangBtn').classList.add('active'); document.getElementById('quickCashLiangBtn').classList.remove('active'); };
    document.getElementById('addCashQuickBtn').onclick = () => {
        const account = document.getElementById('cashCategorySelect').value;
        const amount = parseFloat(document.getElementById('cashAmountInput').value);
        const date = document.getElementById('cashDateInput').value;
        if (!date || isNaN(amount)) { alert("完整填写"); return; }
        addCash({ date, debtor: quickCashPerson, account, amount });
        document.getElementById('cashAmountInput').value = 0;
    };

    // 编辑删除委托
    document.getElementById('debtTbody').addEventListener('click', (e) => {
        if(e.target.classList.contains('edit-debt')){ let id=parseInt(e.target.dataset.id); let d=debts.find(i=>i.id===id); let newAmt=prompt('修改金额', d.amount); if(newAmt) updateDebt(id, parseFloat(newAmt)); }
        if(e.target.classList.contains('del-debt')) if(confirm('删除?')) deleteDebt(parseInt(e.target.dataset.id));
    });
    document.getElementById('cashTbody').addEventListener('click', (e) => {
        if(e.target.classList.contains('edit-cash')){ let id=parseInt(e.target.dataset.id); let c=cashRecords.find(i=>i.id===id); let newAmt=prompt('修改金额', c.amount); if(newAmt) updateCash(id, parseFloat(newAmt)); }
        if(e.target.classList.contains('del-cash')) if(confirm('删除?')) deleteCash(parseInt(e.target.dataset.id));
    });

    // 年份切换
    document.querySelectorAll('.year-btn').forEach(btn => {
        btn.addEventListener('click', () => {
            document.querySelectorAll('.year-btn').forEach(b=>b.classList.remove('active'));
            btn.classList.add('active');
            currentYear = btn.dataset.year;
            renderAllCharts();
        });
    });

    // 密码解锁
    const overlay = document.getElementById('passwordOverlay');
    const unlockBtn = document.getElementById('unlockBtn');
    const pwdInput = document.getElementById('passwordInput');
    const mainApp = document.getElementById('mainApp');
    function unlock() { if(pwdInput.value === "864456"){ overlay.style.display='none'; mainApp.style.display='block'; loadLocal(); } else { document.getElementById('pwdError').innerText='密码错误'; pwdInput.value=''; } }
    unlockBtn.onclick = unlock;
    pwdInput.addEventListener('keypress', e => { if(e.key === 'Enter') unlock(); });
</script>
</body>
</html>
