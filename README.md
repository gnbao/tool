<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>项目收益分配可视化计算工具</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#1e40af',
                        secondary: '#3b82f6',
                        accent: '#93c5fd',
                        neutral: '#f1f5f9',
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .table-scroll {
                overflow-x: auto;
            }
            .input-card {
                @apply bg-white rounded-lg shadow-md p-4 mb-4 transition-all hover:shadow-lg;
            }
            .btn-primary {
                @apply bg-primary text-white px-4 py-2 rounded-md hover:bg-primary/90 transition-colors;
            }
            .btn-secondary {
                @apply bg-gray-200 text-gray-800 px-4 py-2 rounded-md hover:bg-gray-300 transition-colors;
            }
            /* Adjusted chart container styles */
            .chart-container {
                @apply relative h-[50vh]; /* Set height relative to viewport height for mobile responsiveness */
            }
            .chart-container > canvas {
                @apply w-full h-full; /* The canvas fills the container without absolute positioning */
            }
        }
    </style>
</head>
<body class="bg-gray-50 font-sans">
    <div class="container mx-auto px-4 py-8 max-w-7xl">
        <header class="mb-8 text-center">
            <h1 class="text-[clamp(1.8rem,3vw,2.5rem)] font-bold text-primary mb-2">项目收益分配可视化计算工具</h1>
            <p class="text-gray-600">输入参数，自动计算收益分配</p>
        </header>

        <div class="space-y-8">
            <div class="space-y-6">
                <h2 class="text-xl font-semibold text-primary">参数设置</h2>
                
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="space-y-4">
                        <div class="input-card">
                            <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">一、基础参数</h3>
                            <div class="space-y-3">
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">计划合同额（元）</label>
                                    <input type="number" id="contractAmount" value="2600000" min="0" step="5" 
                                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-secondary">
                                </div>
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">预付款（元）</label>
                                    <input type="number" id="advancePayment" value="200000" min="0" step="5" 
                                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-secondary">
                                    <p class="text-xs text-gray-500 mt-1">此金额不计入手术分成表格</p>
                                </div>
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">项目周期（月）</label>
                                    <input type="number" id="cycle" value="48" min="1" step="1" 
                                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-secondary">
                                    <p class="text-xs text-gray-500 mt-1">周期变更后需重新生成/输入月份产值</p>
                                </div>
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">冷静期（月）</label>
                                    <input type="number" id="coolingPeriod" value="1" min="0" step="1" 
                                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-secondary">
                                    <p class="text-xs text-gray-500 mt-1">设为0时，首月即开始计算公司收益</p>
                                </div>
                            </div>
                        </div>

                        <div class="input-card">
                            <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">二、月份产值设置</h3>
                            <div class="space-y-3">
                                <div class="grid grid-cols-3 gap-2">
                                    <div>
                                        <label class="block text-xs font-medium text-gray-700 mb-1">初始产值（元）</label>
                                        <input type="number" id="initialOutput" value="20000" min="5" step="5" 
                                            class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-700 mb-1">月递增额（元）</label>
                                        <input type="number" id="monthlyIncrease" value="10000" min="5" step="5" 
                                            class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-700 mb-1">产值阈值（元）</label>
                                        <input type="number" id="outputThreshold" value="130000" min="5" step="5" 
                                            class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary">
                                    </div>
                                </div>
                                <div class="flex gap-2">
                                    <button id="autoGenerateOutput" class="btn-secondary flex-1 text-sm py-2">自动生成产值（首年递增）</button>
                                    <button id="resetOutput" class="btn-secondary flex-1 text-sm py-2">重置产值</button>
                                </div>
                                <div class="text-xs text-gray-500 mt-1">
                                    说明：数值需为5的整数倍，自动生成逻辑为「首年按递增额增长，达阈值后稳定」
                                </div>
                                
                                <div id="monthlyOutputContainer" class="mt-4 max-h-60 overflow-y-auto pr-1 space-y-2 border border-gray-200 rounded-md p-2">
                                    </div>
                            </div>
                        </div>
                    </div>

                    <div class="space-y-4">
                        <div class="input-card">
                            <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">三、提成比例</h3>
                            <div class="space-y-3">
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">公司销售提成比例（%）</label>
                                    <input type="number" id="companySalesRate" value="5" min="0" max="100" step="0.1" 
                                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-secondary">
                                </div>
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">中间人预付款提成比例（%）</label>
                                    <input type="number" id="middlemanAdvanceRate" value="15" min="0" max="100" step="0.1" 
                                        class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-secondary">
                                </div>
                            </div>
                        </div>

                        <div class="input-card">
                            <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">四、保底期内分成比例</h3>
                             <div id="guaranteeRates" class="grid grid-cols-2 md:grid-cols-4 gap-2">
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">公司（%）</label>
                                    <input type="number" id="guaranteeCompanyRate" value="60" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary guarantee-rate">
                                </div>
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">医院（%）</label>
                                    <input type="number" id="guaranteeHospitalRate" value="20" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary guarantee-rate">
                                </div>
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">医生（%）</label>
                                    <input type="number" id="guaranteeDoctorRate" value="20" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary guarantee-rate">
                                </div>
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">中间人（%）</label>
                                    <input type="number" id="guaranteeMiddlemanRate" value="0" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary guarantee-rate">
                                </div>
                            </div>
                            <div class="text-red-500 text-xs hidden mt-2" id="rateError">⚠️ 保底期内分成比例总和需为100%</div>
                        </div>

                        <div class="input-card">
                            <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">五、公司保底完成后分成比例</h3>
                            <div class="text-xs text-gray-500 mb-2">当公司完成保底后，剩余及后续月份将使用此新比例</div>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-2">
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">公司（%）</label>
                                    <input type="number" id="postGuaranteeCompanyRate" value="0" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary post-guarantee-rate">
                                </div>
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">医院（%）</label>
                                    <input type="number" id="postGuaranteeHospitalRate" value="80" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary post-guarantee-rate">
                                </div>
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">医生（%）</label>
                                    <input type="number" id="postGuaranteeDoctorRate" value="20" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary post-guarantee-rate">
                                </div>
                                <div>
                                    <label class="block text-xs font-medium text-gray-700 mb-1">中间人（%）</label>
                                    <input type="number" id="postGuaranteeMiddlemanRate" value="0" min="0" max="100" step="1"
                                        class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary post-guarantee-rate">
                                </div>
                            </div>
                            <div class="text-red-500 text-xs hidden mt-2" id="postRateError">⚠️ 保底完成后分成比例总和需为100%</div>
                        </div>

                        <div class="input-card">
                            <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">六、公司比例对比方案</h3>
                            <div class="space-y-2">
                                <div class="grid grid-cols-3 gap-2">
                                    <div>
                                        <label class="block text-xs font-medium text-gray-700 mb-1">方案1（%）</label>
                                        <input type="number" id="compareRate1" value="2" min="0" max="20" step="0.5"
                                            class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-700 mb-1">方案2（%）</label>
                                        <input type="number" id="compareRate2" value="5" min="0" max="20" step="0.5"
                                            class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-700 mb-1">方案3（%）</label>
                                        <input type="number" id="compareRate3" value="8" min="0" max="20" step="0.5"
                                            class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-1 focus:ring-secondary">
                                    </div>
                                </div>
                                <p class="text-xs text-gray-500">说明：基于基础方案，计算公司比例降低对应点数后的收益差异</p>
                            </div>
                        </div>

                        <button id="calculateBtn" class="btn-primary w-full py-3 text-base">开始计算 & 生成报告</button>
                    </div>
                </div>
            </div>

            <div class="space-y-8">
                <h2 class="text-xl font-semibold text-primary">计算结果与分析</h2>

                <div class="bg-white rounded-lg shadow-md p-4">
                    <h3 class="text-lg font-semibold text-primary mb-4 border-b pb-2">关键指标汇总</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                        <div class="border border-gray-200 rounded-lg p-3 text-center">
                            <p class="text-sm text-gray-600 mb-1">总产值（元）</p>
                            <p id="totalOutputValue" class="text-xl font-bold text-primary">5,000,000</p>
                        </div>
                        <div class="border border-gray-200 rounded-lg p-3 text-center">
                            <p class="text-sm text-gray-600 mb-1">公司分成总额（元）</p>
                            <p id="operationShareValue" class="text-xl font-bold text-primary">2,800,000</p>
                        </div>
                        <div class="border border-gray-200 rounded-lg p-3 text-center">
                            <p class="text-sm text-gray-600 mb-1">公司保底（元）</p>
                            <p id="companyGuaranteeValue" class="text-xl font-bold text-primary">3,000,000</p>
                            <p class="text-xs text-gray-500 mt-1">(含预付款)</p>
                        </div>
                        <div class="border border-gray-200 rounded-lg p-3 text-center">
                            <p class="text-sm text-gray-600 mb-1">公司累计收益（元）</p>
                            <p id="companyTotalIncome" class="text-xl font-bold text-primary">3,000,000</p>
                            <p class="text-xs text-gray-500 mt-1">(含预付款)</p>
                        </div>
                    </div>

                    <div class="mt-4 p-3 bg-blue-50 rounded-lg border border-blue-100">
                        <h4 class="text-sm font-medium text-primary mb-1 flex items-center">
                            <i class="fa fa-info-circle mr-2"></i>合同完成情况
                        </h4>
                        <p id="contractCompletionText" class="text-sm text-gray-700">预计在第36个月完成合同额目标</p>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
                        <div class="border border-gray-200 rounded-lg p-3">
                            <h4 class="text-sm font-medium text-gray-700 mb-2">公司销售提成</h4>
                            <div class="grid grid-cols-2 gap-2 text-sm">
                                <p class="text-gray-600">预付款提成（元）</p>
                                <p id="companyAdvanceCommission" class="font-medium">20,000</p>
                                <p class="text-gray-600">手术分成提成（元）</p>
                                <p id="companyOperationCommission" class="font-medium">280,000</p>
                                <p class="text-gray-600">合计提成（元）</p>
                                <p id="companyTotalCommission" class="font-medium text-primary">300,000</p>
                            </div>
                        </div>
                        <div class="border border-gray-200 rounded-lg p-3">
                            <h4 class="text-sm font-medium text-gray-700 mb-2">中间人提成</h4>
                            <div class="grid grid-cols-2 gap-2 text-sm">
                                <p class="text-gray-600">预付款提成（元）</p>
                                <p id="middlemanAdvanceCommission" class="font-medium">40,000</p>
                                <p class="text-gray-600">分成收益（元）</p>
                                <p id="middlemanShareIncome" class="font-medium">0</p>
                                <p class="text-gray-600">合计收益（元）</p>
                                <p id="middlemanTotalIncome" class="font-medium text-primary">40,000</p>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="space-y-6">
                    <div class="bg-white rounded-lg shadow-md p-4">
                        <h3 class="text-base font-semibold text-primary mb-3">各角色累计收益对比</h3>
                        <div class="chart-container">
                            <canvas id="incomeComparisonChart"></canvas>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md p-4">
                        <h3 class="text-base font-semibold text-primary mb-3">月份产值趋势</h3>
                        <div class="chart-container">
                            <canvas id="outputTrendChart"></canvas>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md p-4">
                        <h3 class="text-base font-semibold text-primary mb-3">公司累计收益趋势</h3>
                        <div class="chart-container">
                            <canvas id="companyIncomeTrendChart"></canvas>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md p-4">
                        <h3 class="text-base font-semibold text-primary mb-3">月度收益波动趋势</h3>
                        <div class="chart-container">
                            <canvas id="monthlyIncomeTrendChart"></canvas>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md p-4">
                        <h3 class="text-base font-semibold text-primary mb-3">公司比例对比方案收益差异</h3>
                        <div class="chart-container">
                            <canvas id="planComparisonChart"></canvas>
                        </div>
                    </div>
                </div>

                <div class="bg-white rounded-lg shadow-md p-4">
                    <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">公司比例对比分析</h3>
                    <div class="table-scroll">
                        <table class="min-w-full divide-y divide-gray-200 text-sm">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">对比方案</th>
                                    <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">公司比例降低（点）</th>
                                    <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">最终累计收益（元）</th>
                                    <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">收益差异（元）</th>
                                </tr>
                            </thead>
                            <tbody class="bg-white divide-y divide-gray-200">
                                <tr>
                                    <td class="px-4 py-2">基础方案</td>
                                    <td class="px-4 py-2">0</td>
                                    <td id="basePlanIncome" class="px-4 py-2 font-medium">3,000,000</td>
                                    <td class="px-4 py-2">0</td>
                                </tr>
                                <tr>
                                    <td class="px-4 py-2">方案1</td>
                                    <td id="comparePlan1Rate" class="px-4 py-2">2</td>
                                    <td id="comparePlan1Income" class="px-4 py-2 font-medium">2,900,000</td>
                                    <td id="comparePlan1Diff" class="px-4 py-2 text-red-500">-100,000</td>
                                </tr>
                                <tr>
                                    <td class="px-4 py-2">方案2</td>
                                    <td id="comparePlan2Rate" class="px-4 py-2">5</td>
                                    <td id="comparePlan2Income" class="px-4 py-2 font-medium">2,850,000</td>
                                    <td id="comparePlan2Diff" class="px-4 py-2 text-red-500">-150,000</td>
                                </tr>
                                <tr>
                                    <td class="px-4 py-2">方案3</td>
                                    <td id="comparePlan3Rate" class="px-4 py-2">8</td>
                                    <td id="comparePlan3Income" class="px-4 py-2 font-medium">2,800,000</td>
                                    <td id="comparePlan3Diff" class="px-4 py-2 text-red-500">-200,000</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>

                <div class="bg-white rounded-lg shadow-md p-4">
                    <h3 class="text-lg font-semibold text-primary mb-3 border-b pb-2">详细月份手术分成收益分配表</h3>
                    <p class="text-xs text-gray-500 mb-3">注：此表仅显示手术分成部分，不包含预付款金额</p>
                    <div class="table-scroll">
                        <table class="min-w-full divide-y divide-gray-200 text-sm">
                            <thead class="bg-gray-50 sticky top-0 z-10">
                                <tr>
                                    <th class="px-3 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">月份</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">当月产值（元）</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">公司分成（元）</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">医生分成（元）</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">医院分成（元）</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">中间人分成（元）</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">公司累计分成（元）</th>
                                    <th class="px-3 py-2 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">状态</th>
                                </tr>
                            </thead>
                            <tbody id="detailTableBody" class="bg-white divide-y divide-gray-200">
                                </tbody>
                            <tfoot class="bg-gray-100 font-bold">
                                <tr id="detailTableFooter">
                                    <td class="px-3 py-3 text-left">合计</td>
                                    <td id="totalOutputFooter" class="px-3 py-3 text-right">0</td>
                                    <td id="totalCompanyFooter" class="px-3 py-3 text-right">0</td>
                                    <td id="totalDoctorFooter" class="px-3 py-3 text-right">0</td>
                                    <td id="totalHospitalFooter" class="px-3 py-3 text-right">0</td>
                                    <td id="totalMiddlemanFooter" class="px-3 py-3 text-right">0</td>
                                    <td id="totalCompanyCumulativeFooter" class="px-3 py-3 text-right">0</td>
                                    <td class="px-3 py-3 text-right"></td>
                                </tr>
                            </tfoot>
                        </table>
                    </div>
                    <div class="mt-4 text-center text-gray-500 text-sm" id="tableLoading">点击"开始计算"生成详细数据...</div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 全局变量
        let monthlyOutputData = [];
        let contractCompletionMonth = null;
        let myChart1, myChart2, myChart3, myChart4, myChart5; // Declare chart variables

        // 工具函数
        function formatMoney(num) {
            return num.toLocaleString('zh-CN', { minimumFractionDigits: 0, maximumFractionDigits: 0 });
        }

        function isMultipleOfFive(num) {
            return num % 5 === 0;
        }

        // 验证函数
        function validateGuaranteeRates() {
            const company = parseFloat(document.getElementById('guaranteeCompanyRate').value) || 0;
            const hospital = parseFloat(document.getElementById('guaranteeHospitalRate').value) || 0;
            const doctor = parseFloat(document.getElementById('guaranteeDoctorRate').value) || 0;
            const middleman = parseFloat(document.getElementById('guaranteeMiddlemanRate').value) || 0;
            const total = company + hospital + doctor + middleman;
            const isValid = Math.abs(total - 100) <= 0.1;
            document.getElementById('rateError').classList.toggle('hidden', isValid);
            return isValid;
        }

        function validatePostGuaranteeRates() {
            const company = parseFloat(document.getElementById('postGuaranteeCompanyRate').value) || 0;
            const hospital = parseFloat(document.getElementById('postGuaranteeHospitalRate').value) || 0;
            const doctor = parseFloat(document.getElementById('postGuaranteeDoctorRate').value) || 0;
            const middleman = parseFloat(document.getElementById('postGuaranteeMiddlemanRate').value) || 0;
            const total = company + hospital + doctor + middleman;
            const isValid = Math.abs(total - 100) <= 0.1;
            document.getElementById('postRateError').classList.toggle('hidden', isValid);
            return isValid;
        }

        // UI渲染函数
        function renderMonthlyOutputInputs() {
            const cycle = parseInt(document.getElementById('cycle').value) || 36;
            const container = document.getElementById('monthlyOutputContainer');
            container.innerHTML = '';
            monthlyOutputData = new Array(cycle).fill(0);

            for (let i = 0; i < cycle; i++) {
                const month = i + 1;
                const inputGroup = document.createElement('div');
                inputGroup.className = 'flex items-center gap-2';
                inputGroup.innerHTML = `
                    <label class="w-16 text-xs font-medium text-gray-700">第${month}月</label>
                    <input type="number" class="monthly-output-input flex-1 px-2 py-1 border border-gray-300 rounded-md text-sm"
                        data-month="${month}" min="5" step="5" placeholder="输入产值（5的倍数）">
                    <span class="text-red-500 text-xs hidden">需为5的倍数</span>`;
                container.appendChild(inputGroup);
                const input = inputGroup.querySelector('input');
                const errorSpan = inputGroup.querySelector('span');
                input.addEventListener('input', function() {
                    const value = parseInt(this.value) || 0;
                    if (value > 0 && !isMultipleOfFive(value)) {
                        errorSpan.classList.remove('hidden');
                        this.classList.add('border-red-500');
                    } else {
                        errorSpan.classList.add('hidden');
                        this.classList.remove('border-red-500');
                        monthlyOutputData[i] = value;
                    }
                });
            }
        }

        function autoGenerateMonthlyOutput() {
            const cycle = parseInt(document.getElementById('cycle').value) || 36;
            const initialOutput = parseInt(document.getElementById('initialOutput').value) || 20000;
            const monthlyIncrease = parseInt(document.getElementById('monthlyIncrease').value) || 10000;
            const outputThreshold = parseInt(document.getElementById('outputThreshold').value) || 130000;
            if (![initialOutput, monthlyIncrease, outputThreshold].every(isMultipleOfFive)) {
                alert('初始产值、月递增额、产值阈值需为5的倍数。');
                return;
            }
            let currentOutput = initialOutput;
            document.querySelectorAll('.monthly-output-input').forEach((input, index) => {
                const month = index + 1;
                if (month <= 12 && currentOutput < outputThreshold) {
                    monthlyOutputData[index] = currentOutput;
                    currentOutput = Math.min(currentOutput + monthlyIncrease, outputThreshold);
                } else {
                    monthlyOutputData[index] = outputThreshold;
                }
                input.value = monthlyOutputData[index];
                input.classList.remove('border-red-500');
                input.parentElement.querySelector('span').classList.add('hidden');
            });
        }

        function resetMonthlyOutput() {
            document.querySelectorAll('.monthly-output-input').forEach((input, index) => {
                input.value = '';
                monthlyOutputData[index] = 0;
                input.classList.remove('border-red-500');
                input.parentElement.querySelector('span').classList.add('hidden');
            });
        }
        
        // 核心计算逻辑，可复用于对比方案
        function runSimulation(rates) {
            const contractAmount = parseFloat(document.getElementById('contractAmount').value) || 0;
            const advancePayment = parseFloat(document.getElementById('advancePayment').value) || 0;
            const cycle = parseInt(document.getElementById('cycle').value) || 36;
            const coolingPeriod = parseInt(document.getElementById('coolingPeriod').value) || 0;
            const companyGuaranteeTarget = contractAmount - advancePayment;
            
            const postGuaranteeRates = {
                company: parseFloat(document.getElementById('postGuaranteeCompanyRate').value) / 100 || 0,
                hospital: parseFloat(document.getElementById('postGuaranteeHospitalRate').value) / 100 || 0,
                doctor: parseFloat(document.getElementById('postGuaranteeDoctorRate').value) / 100 || 0,
                middleman: parseFloat(document.getElementById('postGuaranteeMiddlemanRate').value) / 100 || 0
            };

            const monthlyData = [];
            let companyTotal = 0, hospitalTotal = 0, doctorTotal = 0, middlemanTotal = 0;
            let companyGuaranteeReached = false;
            let completionMonth = null;

            for (let month = 1; month <= cycle; month++) {
                const output = monthlyOutputData[month - 1] || 0;
                let companyIncome = 0, doctorIncome = 0, hospitalIncome = 0, middlemanIncome = 0;
                let status = '';
                
                // 检查是否已达到保底
                const isGuaranteeReachedThisMonth = !companyGuaranteeReached && (companyTotal + Math.round(output * rates.company)) >= companyGuaranteeTarget;
                
                if (isGuaranteeReachedThisMonth) {
                    completionMonth = month;
                    companyGuaranteeReached = true;
                    
                    const shortage = companyGuaranteeTarget - companyTotal;
                    companyIncome = shortage;
                    
                    doctorIncome = Math.round(output * rates.doctor);
                    hospitalIncome = Math.round(output * rates.hospital);
                    
                    const remainingAmount = output - companyIncome - doctorIncome - hospitalIncome;
                    if (rates.middleman > 0) {
                        middlemanIncome = remainingAmount;
                    } else {
                        hospitalIncome += remainingAmount; // If middleman rate is 0, leftover goes to hospital
                    }
                    
                    status = '保底完成月';

                } else if (companyGuaranteeReached) {
                    // 保底完成后，使用新的分成比例
                    status = '保底后';
                    companyIncome = Math.round(output * postGuaranteeRates.company);
                    hospitalIncome = Math.round(output * postGuaranteeRates.hospital);
                    doctorIncome = Math.round(output * postGuaranteeRates.doctor);
                    middlemanIncome = Math.round(output * postGuaranteeRates.middleman);
                } else if (month <= coolingPeriod) {
                    // 冷静期内
                    status = '冷静期';
                    companyIncome = 0;
                    middlemanIncome = 0;
                    doctorIncome = Math.round(output * rates.doctor);
                    hospitalIncome = output - doctorIncome;
                } else {
                    // 保底完成前，使用保底期内分成比例
                    status = '保底期内';
                    companyIncome = Math.round(output * rates.company);
                    hospitalIncome = Math.round(output * rates.hospital);
                    doctorIncome = Math.round(output * rates.doctor);
                    middlemanIncome = Math.round(output * rates.middleman);
                }
                
                // 修正因四舍五入产生的差额
                const totalCalculated = companyIncome + hospitalIncome + doctorIncome + middlemanIncome;
                const diff = output - totalCalculated;
                if (diff !== 0) {
                    // The primary recipient of the difference should be the hospital unless otherwise specified
                    hospitalIncome += diff;
                }

                companyTotal += companyIncome;
                hospitalTotal += hospitalIncome;
                doctorTotal += doctorIncome;
                middlemanTotal += middlemanIncome;
                monthlyData.push({ month, output, companyIncome, doctorIncome, hospitalIncome, middlemanIncome, companyTotal, status });
            }
            
            return { monthlyData, companyTotal, hospitalTotal, doctorTotal, middlemanTotal, completionMonth };
        }


        function calculate() {
            if (!validateGuaranteeRates() || !validatePostGuaranteeRates()) return;
            if (!monthlyOutputData.some(v => v > 0)) {
                alert('请先生成或输入月份产值数据');
                return;
            }

            // 获取基础参数
            const contractAmount = parseFloat(document.getElementById('contractAmount').value) || 0;
            const advancePayment = parseFloat(document.getElementById('advancePayment').value) || 0;
            const companySalesRate = parseFloat(document.getElementById('companySalesRate').value) / 100 || 0;
            const middlemanAdvanceRate = parseFloat(document.getElementById('middlemanAdvanceRate').value) / 100 || 0;

            // 基础方案
            const baseRates = {
                company: parseFloat(document.getElementById('guaranteeCompanyRate').value) / 100 || 0,
                hospital: parseFloat(document.getElementById('guaranteeHospitalRate').value) / 100 || 0,
                doctor: parseFloat(document.getElementById('guaranteeDoctorRate').value) / 100 || 0,
                middleman: parseFloat(document.getElementById('guaranteeMiddlemanRate').value) / 100 || 0,
            };
            const baseResult = runSimulation(baseRates);
            
            // 更新UI
            updateUI(baseResult.monthlyData, { contractAmount, advancePayment, companyTotal: baseResult.companyTotal, hospitalTotal: baseResult.hospitalTotal, doctorTotal: baseResult.doctorTotal, middlemanTotal: baseResult.middlemanTotal, companySalesRate, middlemanAdvanceRate, completionMonth: baseResult.completionMonth });
            
            // 更新对比方案表格和图表
            updateComparisonUI(baseResult.companyTotal, baseResult.monthlyData);
            
            const finalCompanyTotalWithAdvance = baseResult.companyTotal + advancePayment;
            const middlemanIncomeTotal = baseResult.middlemanTotal + (advancePayment * middlemanAdvanceRate);
            renderCharts(baseResult.monthlyData, finalCompanyTotalWithAdvance, baseResult.hospitalTotal, baseResult.doctorTotal, middlemanIncomeTotal);
        }

        function updateComparisonUI(baseCompanyTotal, baseMonthlyData) {
            const advancePayment = parseFloat(document.getElementById('advancePayment').value) || 0;
            const coolingPeriod = parseInt(document.getElementById('coolingPeriod').value) || 0;
            const baseIncome = baseCompanyTotal + advancePayment;
            
            document.getElementById('basePlanIncome').textContent = formatMoney(baseIncome);
            const totalOutputAfterCooling = baseMonthlyData.slice(coolingPeriod).reduce((sum, d) => sum + d.output, 0);

            ['1', '2', '3'].forEach((i, index) => {
                const decreasePoints = parseFloat(document.getElementById(`compareRate${i}`).value) || 0;
                const theoreticalLoss = totalOutputAfterCooling * (decreasePoints / 100);
                const planIncome = Math.max(0, baseIncome - theoreticalLoss);
                
                const diff = planIncome - baseIncome;
                
                document.getElementById(`comparePlan${i}Rate`).textContent = decreasePoints;
                document.getElementById(`comparePlan${i}Income`).textContent = formatMoney(planIncome);
                const diffElem = document.getElementById(`comparePlan${i}Diff`);
                diffElem.textContent = (diff >= 0 ? '+' : '') + formatMoney(diff);
                diffElem.className = diff >= 0 ? 'px-4 py-2 text-green-500' : 'px-4 py-2 text-red-500';
            });
        }
        
        // 更新UI主函数
        function updateUI(monthlyData, params) {
            const { contractAmount, advancePayment, companyTotal, hospitalTotal, doctorTotal, middlemanTotal, companySalesRate, middlemanAdvanceRate, completionMonth } = params;

            // 更新完成情况
            const completionText = document.getElementById('contractCompletionText');
            if (completionMonth) {
                completionText.textContent = `公司保底于第 ${completionMonth} 个月完成，后续收益按新规则分配。`;
            } else {
                completionText.textContent = `在整个周期内未完成公司保底目标。`;
            }

            // 更新关键指标
            const finalCompanyTotalWithAdvance = companyTotal + advancePayment;
            document.getElementById('totalOutputValue').textContent = formatMoney(monthlyData.reduce((sum, d) => sum + d.output, 0));
            document.getElementById('operationShareValue').textContent = formatMoney(companyTotal);
            document.getElementById('companyGuaranteeValue').textContent = formatMoney(contractAmount);
            document.getElementById('companyTotalIncome').textContent = formatMoney(finalCompanyTotalWithAdvance);

            // 更新提成
            const companyAdvanceCommission = Math.round(advancePayment * companySalesRate);
            const companyOperationCommission = Math.round(companyTotal * companySalesRate);
            document.getElementById('companyAdvanceCommission').textContent = formatMoney(companyAdvanceCommission);
            document.getElementById('companyOperationCommission').textContent = formatMoney(companyOperationCommission);
            document.getElementById('companyTotalCommission').textContent = formatMoney(companyAdvanceCommission + companyOperationCommission);

            const middlemanAdvanceCommission = Math.round(advancePayment * middlemanAdvanceRate);
            document.getElementById('middlemanAdvanceCommission').textContent = formatMoney(middlemanAdvanceCommission);
            document.getElementById('middlemanShareIncome').textContent = formatMoney(middlemanTotal);
            document.getElementById('middlemanTotalIncome').textContent = formatMoney(middlemanTotal + middlemanAdvanceCommission);

            // 更新详细表格
            const tableBody = document.getElementById('detailTableBody');
            tableBody.innerHTML = '';
            document.getElementById('tableLoading').classList.add('hidden');
            let cumCompany = 0;
            monthlyData.forEach(data => {
                cumCompany += data.companyIncome;
                const row = document.createElement('tr');
                if (data.status.includes('保底后')) row.className = 'bg-green-50';
                else if (data.status === '冷静期') row.className = 'bg-yellow-50';
                else if (data.status.includes('完成')) row.className = 'bg-blue-50';
                row.innerHTML = `
                    <td class="px-3 py-2">${data.month}</td>
                    <td class="px-3 py-2 text-right">${formatMoney(data.output)}</td>
                    <td class="px-3 py-2 text-right">${formatMoney(data.companyIncome)}</td>
                    <td class="px-3 py-2 text-right">${formatMoney(data.doctorIncome)}</td>
                    <td class="px-3 py-2 text-right">${formatMoney(data.hospitalIncome)}</td>
                    <td class="px-3 py-2 text-right">${formatMoney(data.middlemanIncome)}</td>
                    <td class="px-3 py-2 text-right font-medium">${formatMoney(cumCompany)}</td>
                    <td class="px-3 py-2 text-center">
                        <span class="px-2 py-0.5 text-xs rounded-full ${
                            data.status === '冷静期' ? 'bg-yellow-100 text-yellow-800' : 
                            data.status.includes('保底后') ? 'bg-green-100 text-green-800' : 
                            'bg-blue-100 text-blue-800'
                        }">${data.status}</span>
                    </td>`;
                tableBody.appendChild(row);
            });
            document.getElementById('totalOutputFooter').textContent = formatMoney(monthlyData.reduce((s,d) => s + d.output, 0));
            document.getElementById('totalCompanyFooter').textContent = formatMoney(companyTotal);
            document.getElementById('totalDoctorFooter').textContent = formatMoney(doctorTotal);
            document.getElementById('totalHospitalFooter').textContent = formatMoney(hospitalTotal);
            document.getElementById('totalMiddlemanFooter').textContent = formatMoney(middlemanTotal);
            document.getElementById('totalCompanyCumulativeFooter').textContent = formatMoney(companyTotal);
        }
        
        // 渲染图表
        function renderCharts(monthlyData, companyTotalWithAdvance, hospitalTotal, doctorTotal, middlemanTotalWithAdvance) {
            // 图表通用配置
            const commonOptions = { 
                responsive: true,
                maintainAspectRatio: false, 
                scales: {
                    y: { beginAtZero: true },
                    x: { ticks: { autoSkip: true, maxTicksLimit: 20 } }
                },
                plugins: {
                    legend: { position: 'top' },
                    tooltip: {
                        callbacks: {
                            label: function(context) {
                                let label = context.dataset.label || '';
                                if (label) { label += ': '; }
                                if (context.parsed.y !== null) { label += formatMoney(context.parsed.y); }
                                return label;
                            }
                        }
                    }
                }
            };
            
            // 1. 累计收益对比图
            const incomeCtx = document.getElementById('incomeComparisonChart').getContext('2d');
            if (window.incomeChart) window.incomeChart.destroy();
            window.incomeChart = new Chart(incomeCtx, {
                type: 'bar',
                data: {
                    labels: ['公司', '医院', '医生', '中间人'],
                    datasets: [{
                        label: '累计收益（含预付款）',
                        data: [companyTotalWithAdvance, hospitalTotal, doctorTotal, middlemanTotalWithAdvance],
                        backgroundColor: ['#1e40af', '#3b82f6', '#93c5fd', '#bfdbfe'],
                    }]
                },
                options: { ...commonOptions, scales: {y: { beginAtZero: true }} }
            });

            // 2. 产值趋势图
            const trendCtx = document.getElementById('outputTrendChart').getContext('2d');
            if (window.trendChart) window.trendChart.destroy();
            window.trendChart = new Chart(trendCtx, {
                type: 'line',
                data: {
                    labels: monthlyData.map(d => d.month),
                    datasets: [{ label: '当月产值', data: monthlyData.map(d => d.output), borderColor: '#1e40af' }]
                },
                options: { ...commonOptions }
            });

            // 3. 公司累计收益趋势图
            const companyTrendCtx = document.getElementById('companyIncomeTrendChart').getContext('2d');
            if (window.companyTrendChart) window.companyTrendChart.destroy();
            window.companyTrendChart = new Chart(companyTrendCtx, {
                type: 'line',
                data: {
                    labels: monthlyData.map(d => d.month),
                    datasets: [
                        { label: '公司累计收益', data: monthlyData.map(d => d.companyTotal + (parseFloat(document.getElementById('advancePayment').value) || 0)), borderColor: '#1e40af' },
                        { label: '公司保底目标', data: Array(monthlyData.length).fill(parseFloat(document.getElementById('contractAmount').value) || 0), borderColor: '#ef4444', borderDash: [5, 5] }
                    ]
                },
                options: { ...commonOptions }
            });

            // 4. 月度收益波动图
            const monthlyTrendCtx = document.getElementById('monthlyIncomeTrendChart').getContext('2d');
            if (window.monthlyTrendChart) window.monthlyTrendChart.destroy();
            window.monthlyTrendChart = new Chart(monthlyTrendCtx, {
                type: 'line',
                data: {
                    labels: monthlyData.map(d => d.month),
                    datasets: [
                        { label: '公司', data: monthlyData.map(d => d.companyIncome), borderColor: '#1e40af' },
                        { label: '医院', data: monthlyData.map(d => d.hospitalIncome), borderColor: '#3b82f6' },
                        { label: '医生', data: monthlyData.map(d => d.doctorIncome), borderColor: '#93c5fd' },
                        { label: '中间人', data: monthlyData.map(d => d.middlemanIncome), borderColor: '#bfdbfe' },
                    ]
                },
                options: { ...commonOptions }
            });

            // 5. 对比方案图
            const planCtx = document.getElementById('planComparisonChart').getContext('2d');
            if (window.planChart) window.planChart.destroy();
            const baseIncome = monthlyData[monthlyData.length - 1].companyTotal + (parseFloat(document.getElementById('advancePayment').value) || 0);
            const planIncomes = [1,2,3].map(i => {
                const decreasePoints = parseFloat(document.getElementById(`compareRate${i}`).value) || 0;
                const totalOutputAfterCooling = monthlyData.slice(parseInt(document.getElementById('coolingPeriod').value) || 0).reduce((sum, d) => sum + d.output, 0);
                const theoreticalLoss = totalOutputAfterCooling * (decreasePoints / 100);
                return Math.max(0, baseIncome - theoreticalLoss);
            });

            window.planChart = new Chart(planCtx, {
                type: 'bar',
                data: {
                    labels: ['基础方案', '方案1', '方案2', '方案3'],
                    datasets: [{
                        label: '公司累计收益',
                        data: [baseIncome, ...planIncomes],
                        backgroundColor: ['#1e40af', '#3b82f6', '#60a5fa', '#93c5fd']
                    }]
                },
                options: { ...commonOptions, scales: { y: { beginAtZero: true } } }
            });
        }

        // 页面加载与事件绑定
        document.addEventListener('DOMContentLoaded', function() {
            renderMonthlyOutputInputs();
            document.getElementById('cycle').addEventListener('change', () => {
                if (confirm('周期变更将重置产值，是否继续？')) renderMonthlyOutputInputs();
                else this.value = monthlyOutputData.length;
            });
            document.getElementById('autoGenerateOutput').addEventListener('click', autoGenerateMonthlyOutput);
            document.getElementById('resetOutput').addEventListener('click', resetMonthlyOutput);
            document.querySelectorAll('.guarantee-rate').forEach(input => input.addEventListener('input', validateGuaranteeRates));
            document.querySelectorAll('.post-guarantee-rate').forEach(input => input.addEventListener('input', validatePostGuaranteeRates));
            document.getElementById('calculateBtn').addEventListener('click', calculate);
            
            autoGenerateMonthlyOutput();
            calculate();
        });
    </script>
</body>
</html>
