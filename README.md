# f003_cov
f003_疫情传染病数据可视化vue+flask+mysql

编号:F003

> 学长的微信：mmdsj186011  【开发不易、源码有偿、谢谢理解】
>欢迎关注B站

✅  vue + flask 前后端分离架构
✅  实现中国地图、柱状图、折线图、水地图、环图等多种图形的echarts可视化分析
## 视频

[video(video-LQCn5uDL-1755570559380)(type-bilibili)(url-https://player.bilibili.com/player.html?aid=979923756)(image-https://i-blog.csdnimg.cn/img_convert/30868c37501e354ecb77bfc2370ce5a1.png)(title-vue+flask+爬虫 新冠疫情大屏实现 python 可视化分析项目源码)]

## 1 系统功能
1. 读取腾讯新闻api数据解析后，存储到mysql数据库
  2.利用Flask的模板引擎，通过Echats 设计了中国疫情地图、折线图、柱状图等来分析疫情数据，进行数据的可视化
## 2 功能图 & 架构图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5ceb66b34d8443eb84168bf0f790d7ef.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3da14bb710f2406b97257705601b7378.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/497bfb020da34ee1b6bd6f3d92b3672d.jpeg)


## 3 开发环境和关键技术

- WebStorm
- pymysql
- flask
- request 、 json 等

## 源码

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>传染病数据监控大屏</title>
    <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://unpkg.com/element-ui/lib/index.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.2/dist/echarts.min.js"></script>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Microsoft YaHei', 'Segoe UI', sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #0c1b33, #1a3a5f);
            color: #e0f0ff;
            overflow: hidden;
        }
        
        #app {
            padding: 20px;
            position: relative;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        .dashboard-header {
            position: relative;
            margin-bottom: 20px;
            padding: 15px 0;
            border-bottom: 1px solid rgba(64, 158, 255, 0.3);
        }
        
        .dashboard-title {
            text-align: center;
            font-size: 36px;
            font-weight: bold;
            letter-spacing: 4px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 0 10px rgba(79, 172, 254, 0.3);
            margin-bottom: 10px;
        }
        
        .last-update {
            text-align: center;
            font-size: 14px;
            color: #a0d8ff;
        }
        
        .dashboard-grid {
            display: grid;
            grid-template-columns: 1.5fr 1fr;
            grid-template-rows: 1fr 1.5fr 1fr;
            gap: 20px;
            height: calc(100vh - 140px);
            grid-template-areas:
                "map kpi"
                "map trend"
                "dist data";
        }
        
        .dashboard-card {
            background: rgba(16, 42, 88, 0.45);
            border: 1px solid rgba(64, 158, 255, 0.2);
            border-radius: 8px;
            padding: 15px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.18);
            backdrop-filter: blur(4px);
            position: relative;
            overflow: hidden;
        }
        
        .card-title {
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 1px solid rgba(64, 158, 255, 0.2);
            display: flex;
            align-items: center;
        }
        
        .card-title::before {
            content: '';
            display: inline-block;
            width: 4px;
            height: 18px;
            background: #409eff;
            margin-right: 8px;
            border-radius: 2px;
        }
        
        /* 指标卡片样式 */
        .kpi-container {
            grid-area: kpi;
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            grid-template-rows: repeat(2, 1fr);
            gap: 15px;
        }
        
        .kpi-card {
            background: rgba(16, 42, 88, 0.7);
            border-radius: 6px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            position: relative;
            overflow: hidden;
            transition: all 0.3s ease;
        }
        
        .kpi-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
        }
        
        .kpi-label {
            font-size: 14px;
            color: #a0d8ff;
            margin-bottom: 10px;
        }
        
        .kpi-value {
            font-size: 32px;
            font-weight: bold;
            margin-bottom: 5px;
        }
        
        .kpi-change {
            font-size: 13px;
            display: flex;
            align-items: center;
        }
        
        .kpi-change.up {
            color: #f56c6c;
        }
        
        .kpi-change.down {
            color: #67c23a;
        }
        
        /* 地图区域 */
        .map-container {
            grid-area: map;
            position: relative;
        }
        
        /* 趋势图表 */
        .trend-container {
            grid-area: trend;
        }
        
        /* 分布图表 */
        .distribution-container {
            grid-area: dist;
        }
        
        /* 实时数据 */
        .realtime-container {
            grid-area: data;
            display: flex;
            flex-direction: column;
        }
        
        .data-list {
            flex: 1;
            overflow-y: auto;
        }
        
        .data-item {
            padding: 12px 15px;
            border-bottom: 1px solid rgba(64, 158, 255, 0.1);
            display: flex;
            justify-content: space-between;
            transition: background 0.2s;
        }
        
        .data-item:hover {
            background: rgba(64, 158, 255, 0.1);
        }
        
        .province {
            font-weight: 600;
        }
        
        .number {
            color: #f56c6c;
            font-weight: bold;
        }
        
        /* 图表容器 */
        .chart-container {
            width: 100%;
            height: calc(100% - 40px);
            position: relative;
        }
        
        /* 装饰元素 */
        .corner-decoration {
            position: absolute;
            width: 15px;
            height: 15px;
        }
        
        .corner-tl {
            top: 0;
            left: 0;
            border-top: 2px solid #409eff;
            border-left: 2px solid #409eff;
            border-top-left-radius: 4px;
        }
        
        .corner-tr {
            top: 0;
            right: 0;
            border-top: 2px solid #409eff;
            border-right: 2px solid #409eff;
            border-top-right-radius: 4px;
        }
        
        .corner-bl {
            bottom: 0;
            left: 0;
            border-bottom: 2px solid #409eff;
            border-left: 2px solid #409eff;
            border-bottom-left-radius: 4px;
        }
        
        .corner-br {
            bottom: 0;
            right: 0;
            border-bottom: 2px solid #409eff;
            border-right: 2px solid #409eff;
            border-bottom-right-radius: 4px;
        }
        
        /* 响应式调整 */
        @media (max-width: 1200px) {
            .dashboard-grid {
                grid-template-columns: 1fr 1fr;
                grid-template-areas:
                    "map kpi"
                    "trend trend"
                    "dist data";
            }
        }
    </style>
</head>
<body>
    <div id="app">
        <div class="dashboard-header">
            <h1 class="dashboard-title">传染病数据监控大屏</h1>
            <p class="last-update">最后更新: {{ updateTime }}</p>
        </div>
        
        <div class="dashboard-grid">
            <!-- 关键指标区域 -->
            <div class="kpi-container dashboard-card">
                <div class="kpi-card">
                    <div class="corner-decoration corner-tl"></div>
                    <div class="corner-decoration corner-tr"></div>
                    <div class="corner-decoration corner-bl"></div>
                    <div class="corner-decoration corner-br"></div>
                    <div class="kpi-label">累计确诊病例</div>
                    <div class="kpi-value">{{ kpiData.totalCases | formatNumber }}</div>
                    <div class="kpi-change up">
                        <i class="el-icon-caret-top" style="margin-right: 4px;"></i>
                        {{ kpiData.newCases | formatNumber }} 较昨日
                    </div>
                </div>
                
                <div class="kpi-card">
                    <div class="corner-decoration corner-tl"></div>
                    <div class="corner-decoration corner-tr"></div>
                    <div class="corner-decoration corner-bl"></div>
                    <div class="corner-decoration corner-br"></div>
                    <div class="kpi-label">现有确诊病例</div>
                    <div class="kpi-value">{{ kpiData.currentCases | formatNumber }}</div>
                    <div class="kpi-change down">
                        <i class="el-icon-caret-bottom" style="margin-right: 4px;"></i>
                        {{ kpiData.currentChange | formatNumber }} 较昨日
                    </div>
                </div>
                
                <div class="kpi-card">
                    <div class="corner-decoration corner-tl"></div>
                    <div class="corner-decoration corner-tr"></div>
                    <div class="corner-decoration corner-bl"></div>
                    <div class="corner-decoration corner-br"></div>
                    <div class="kpi-label">累计治愈病例</div>
                    <div class="kpi-value">{{ kpiData.totalRecovered | formatNumber }}</div>
                    <div class="kpi-change up">
                        <i class="el-icon-caret-top" style="margin-right: 4px;"></i>
                        {{ kpiData.newRecovered | formatNumber }} 较昨日
                    </div>
                </div>
                
                <div class="kpi-card">
                    <div class="corner-decoration corner-tl"></div>
                    <div class="corner-decoration corner-tr"></div>
                    <div class="corner-decoration corner-bl"></div>
                    <div class="corner-decoration corner-br"></div>
                    <div class="kpi-label">累计死亡病例</div>
                    <div class="kpi-value">{{ kpiData.totalDeaths | formatNumber }}</div>
                    <div class="kpi-change up">
                        <i class="el-icon-caret-top" style="margin-right: 4px;"></i>
                        {{ kpiData.newDeaths | formatNumber }} 较昨日
                    </div>
                </div>
            </div>
            
            <!-- 疫情地图区域 -->
            <div class="map-container dashboard-card">
                <div class="card-title">全国疫情分布地图</div>
                <div class="chart-container" id="map-chart"></div>
            </div>
            
            <!-- 趋势图表区域 -->
            <div class="trend-container dashboard-card">
                <div class="card-title">疫情发展趋势</div>
                <div class="chart-container" id="trend-chart"></div>
            </div>
            
            <!-- 分布图表区域 -->
            <div class="distribution-container dashboard-card">
                <div class="card-title">风险地区分布</div>
                <div class="chart-container" id="distribution-chart"></div>
            </div>
            
            <!-- 实时数据区域 -->
            <div class="realtime-container dashboard-card">
                <div class="card-title">高风险地区实时数据</div>
                <div class="data-list">
                    <div v-for="(item, index) in realtimeData" :key="index" class="data-item">
                        <span class="province">{{ item.province }}</span>
                        <span class="city">{{ item.city }}</span>
                        <span class="number">{{ item.cases }}例</span>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 初始化Vue应用
        new Vue({
            el: '#app',
            data() {
                return {
                    updateTime: this.getCurrentTime(),
                    kpiData: {
                        totalCases: 985265,
                        newCases: 3426,
                        currentCases: 124823,
                        currentChange: -1584,
                        totalRecovered: 846321,
                        newRecovered: 4852,
                        totalDeaths: 14121,
                        newDeaths: 28
                    },
                    realtimeData: [
                        { province: '北京市', city: '朝阳区', cases: 256 },
                        { province: '上海市', city: '浦东新区', cases: 184 },
                        { province: '广东省', city: '深圳市', cases: 152 },
                        { province: '江苏省', city: '南京市', cases: 98 },
                        { province: '浙江省', city: '杭州市', cases: 87 },
                        { province: '湖北省', city: '武汉市', cases: 76 },
                        { province: '四川省', city: '成都市', cases: 65 },
                        { province: '陕西省', city: '西安市', cases: 58 },
                        { province: '天津市', city: '滨海新区', cases: 49 },
                        { province: '辽宁省', city: '沈阳市', cases: 42 }
                    ],
                    mapChart: null,
                    trendChart: null,
                    distributionChart: null
                }
            },
            filters: {
                formatNumber(value) {
                    return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
                }
            },
            mounted() {
                this.initCharts();
                this.updateTimeInterval = setInterval(() => {
                    this.updateTime = this.getCurrentTime();
                }, 60000);
                
                // 模拟实时数据更新
                setInterval(this.updateRealtimeData, 5000);
            },
            beforeDestroy() {
                clearInterval(this.updateTimeInterval);
            },
            methods: {
                getCurrentTime() {
                    const now = new Date();
                    return now.getFullYear() + '-' + 
                           (now.getMonth()+1).toString().padStart(2, '0') + '-' + 
                           now.getDate().toString().padStart(2, '0') + ' ' + 
                           now.getHours().toString().padStart(2, '0') + ':' + 
                           now.getMinutes().toString().padStart(2, '0');
                },
                initCharts() {
                    // 初始化地图图表
                    this.mapChart = echarts.init(document.getElementById('map-chart'));
                    this.renderMap();
                    
                    // 初始化趋势图表
                    this.trendChart = echarts.init(document.getElementById('trend-chart'));
                    this.renderTrendChart();
                    
                    // 初始化分布图表
                    this.distributionChart = echarts.init(document.getElementById('distribution-chart'));
                    this.renderDistributionChart();
                },
                renderMap() {
                    // 地图配置 - 使用模拟数据
                    const option = {
                        tooltip: {
                            trigger: 'item',
                            formatter: '{b}: {c} 例'
                        },
                        visualMap: {
                            min: 0,
                            max: 5000,
                            text: ['高', '低'],
                            realtime: false,
                            calculable: true,
                            inRange: {
                                color: ['#1d7dc2', '#1a7fd5', '#1a6aef', '#1a54ff', '#313bff', '#5c32ff', '#8529ff', '#af1fff', '#d815ff', '#ff0be5']
                            }
                        },
                        series: [
                            {
                                name: '确诊病例',
                                type: 'map',
                                map: 'china',
                                roam: true,
                                label: {
                                    show: true,
                                    fontSize: 10
                                },
                                data: [
                                    {name: '北京', value: 4521},
                                    {name: '天津', value: 1420},
                                    {name: '上海', value: 3845},
                                    {name: '重庆', value: 2641},
                                    {name: '河北', value: 1847},
                                    {name: '河南', value: 2715},
                                    {name: '云南', value: 1218},
                                    {name: '辽宁', value: 1975},
                                    {name: '黑龙江', value: 2140},
                                    {name: '湖南', value: 2310},
                                    {name: '安徽', value: 1645},
                                    {name: '山东', value: 2950},
                                    {name: '新疆', value: 780},
                                    {name: '江苏', value: 3540},
                                    {name: '浙江', value: 3985},
                                    {name: '江西', value: 1470},
                                    {name: '湖北', value: 3760},
                                    {name: '广西', value: 1065},
                                    {name: '甘肃', value: 885},
                                    {name: '山西', value: 1267},
                                    {name: '内蒙古', value: 985},
                                    {name: '陕西', value: 1740},
                                    {name: '吉林', value: 1124},
                                    {name: '福建', value: 2310},
                                    {name: '贵州', value: 1080},
                                    {name: '广东', value: 4210},
                                    {name: '青海', value: 320},
                                    {name: '西藏', value: 85},
                                    {name: '四川', value: 2870},
                                    {name: '宁夏', value: 630},
                                    {name: '海南', value: 780},
                                ]
                            }
                        ]
                    };
                    
                    // 使用中国地图
                    fetch('https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json')
                        .then(response => response.json())
                        .then(chinaJson => {
                            echarts.registerMap('china', chinaJson);
                            this.mapChart.setOption(option);
                        });
                },
                renderTrendChart() {
                    // 趋势图表配置
                    const option = {
                        tooltip: {
                            trigger: 'axis'
                        },
                        legend: {
                            data: ['新增确诊', '现有确诊', '治愈病例'],
                            textStyle: {
                                color: '#e0f0ff'
                            },
                            right: 10,
                            top: 10
                        },
                        grid: {
                            left: '3%',
                            right: '4%',
                            bottom: '3%',
                            containLabel: true
                        },
                        xAxis: {
                            type: 'category',
                            boundaryGap: false,
                            data: ['8/1', '8/2', '8/3', '8/4', '8/5', '8/6', '8/7'],
                            axisLine: {
                                lineStyle: {
                                    color: '#e0f0ff'
                                }
                            }
                        },
                        yAxis: {
                            type: 'value',
                            axisLine: {
                                lineStyle: {
                                    color: '#e0f0ff'
                                }
                            },
                            splitLine: {
                                lineStyle: {
                                    color: 'rgba(255, 255, 255, 0.1)'
                                }
                            }
                        },
                        series: [
                            {
                                name: '新增确诊',
                                type: 'line',
                                data: [4250, 4870, 4520, 3920, 3560, 3420, 3426],
                                smooth: true,
                                lineStyle: {
                                    width: 3,
                                    color: '#f56c6c'
                                },
                                symbol: 'circle',
                                symbolSize: 8,
                                itemStyle: {
                                    color: '#f56c6c'
                                },
                                areaStyle: {
                                    color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                                        { offset: 0, color: 'rgba(245, 108, 108, 0.5)' },
                                        { offset: 1, color: 'rgba(245, 108, 108, 0.1)' }
                                    ])
                                }
                            },
                            {
                                name: '现有确诊',
                                type: 'line',
                                data: [154200, 148600, 142500, 135800, 129500, 126400, 124800],
                                smooth: true,
                                lineStyle: {
                                    width: 3,
                                    color: '#e6a23c'
                                },
                                symbol: 'circle',
                                symbolSize: 8,
                                itemStyle: {
                                    color: '#e6a23c'
                                }
                            },
                            {
                                name: '治愈病例',
                                type: 'line',
                                data: [785240, 795680, 805420, 815650, 825900, 836580, 846320],
                                smooth: true,
                                lineStyle: {
                                    width: 3,
                                    color: '#67c23a'
                                },
                                symbol: 'circle',
                                symbolSize: 8,
                                itemStyle: {
                                    color: '#67c23a'
                                }
                            }
                        ]
                    };
                    this.trendChart.setOption(option);
                },
                renderDistributionChart() {
                    // 分布图表配置
                    const option = {
                        tooltip: {
                            trigger: 'item',
                            formatter: '{a} <br/>{b}: {c} ({d}%)'
                        },
                        legend: {
                            orient: 'vertical',
                            left: 10,
                            top: 'center',
                            data: ['高风险地区', '中风险地区', '低风险地区'],
                            textStyle: {
                                color: '#e0f0ff'
                            }
                        },
                        series: [
                            {
                                name: '风险地区分布',
                                type: 'pie',
                                radius: ['50%', '70%'],
                                center: ['65%', '50%'],
                                avoidLabelOverlap: false,
                                itemStyle: {
                                    borderRadius: 10,
                                    borderColor: '#0c1b33',
                                    borderWidth: 2
                                },
                                label: {
                                    show: false,
                                    position: 'center'
                                },
                                emphasis: {
                                    label: {
                                        show: true,
                                        fontSize: '16',
                                        fontWeight: 'bold',
                                        color: '#fff'
                                    }
                                },
                                labelLine: {
                                    show: false
                                },
                                data: [
                                    { value: 25, name: '高风险地区', itemStyle: { color: '#f56c6c' } },
                                    { value: 45, name: '中风险地区', itemStyle: { color: '#e6a23c' } },
                                    { value: 30, name: '低风险地区', itemStyle: { color: '#67c23a' } }
                                ]
                            }
                        ]
                    };
                    this.distributionChart.setOption(option);
                },
                updateRealtimeData() {
                    // 模拟实时数据更新
                    const provinces = ['北京市', '上海市', '广东省', '江苏省', '浙江省', '湖北省', '四川省', '陕西省', '天津市', '辽宁省'];
                    const cities = {
                        '北京市': ['朝阳区', '海淀区', '丰台区', '西城区', '东城区'],
                        '上海市': ['浦东新区', '闵行区', '徐汇区', '黄浦区', '虹口区'],
                        '广东省': ['深圳市', '广州市', '东莞市', '佛山市', '珠海市'],
                        '江苏省': ['南京市', '苏州市', '无锡市', '徐州市', '常州市'],
                        '浙江省': ['杭州市', '宁波市', '温州市', '嘉兴市', '绍兴市'],
                        '湖北省': ['武汉市', '黄冈市', '孝感市', '荆州市', '宜昌市'],
                        '四川省': ['成都市', '绵阳市', '德阳市', '泸州市', '南充市'],
                        '陕西省': ['西安市', '咸阳市', '宝鸡市', '渭南市', '汉中市'],
                        '天津市': ['滨海新区', '南开区', '河西区', '河东区', '河北区'],
                        '辽宁省': ['沈阳市', '大连市', '鞍山市', '抚顺市', '锦州市']
                    };
                    
                    const newData = [];
                    for (let i = 0; i < 10; i++) {
                        const province = provinces[Math.floor(Math.random() * provinces.length)];
                        const cityArr = cities[province];
                        const city = cityArr[Math.floor(Math.random() * cityArr.length)];
                        const cases = Math.floor(Math.random() * 100) + 20;
                        newData.push({ province, city, cases });
                    }
                    
                    this.realtimeData = newData;
                }
            }
        });
    </script>
</body>
</html>

```

