
# NXpower + MGMS融合方案

## 1. 业务场景
```mermaid
flowchart LR
    subgraph 输入数据
        subgraph 系统配置数据
            A1[地理位置（经纬度、站点编号）]
            A2[天气数据（温度、湿度、风速、辐射等）]
            A3[光伏系统信息（装机容量、逆变器参数）]
            A4[储能系统参数（容量、SOC、充放电效率）]
            A5[电价模型（实时电价、分时电价、政策）]
            A8[环境特征（空气质量、地形、植被等）]
        end
        subgraph 系统历史数据
            A6[负荷历史数据（用电曲线、峰谷负荷）]
            A7[光伏历史数据（发电量、功率曲线）]        
        end
        subgraph 采集数据
            B1[电网连接点总表电度和功率]
            B2[电负荷总电度和功率]
            B3[光伏发电功率]
            B4[储能充放功率和SoC]
        end
    end

    subgraph 业务处理与AI优化
        C1[MGMS微网系统配置]
        C2[数据采集与规约转换]
        C3[NXpower软件系统]
        C4[MGMS AI策略优化器]
    end

    subgraph 输出结果
        D1[天气预测]
        D2[负荷预测]
        D3[光伏发电量预测]
        D4[储能充放策略优化建议]
        D5[经济性分析（成本、收益、峰谷套利）]
        D6[设备运行状态监控（SOC、功率等）]
        D7[能量流动与调度方案]
    end

    A1 --> C1
    A2 --> C1
    A3 --> C1
    A4 --> C1
    A5 --> C1
    A6 --> C3
    A7 --> C3
    A8 --> C1
    B1 --> C2
    B2 --> C2
    B3 --> C2
    B4 --> C2
    C1 --> C4
    C2 --> C3
    C3 --> C4
    C4 --> D1
    C4 --> D2
    C4 --> D3
    C4 --> D4
    C4 --> D5
    C4 --> D6
    C4 --> D7


```

---

// Mermaid图说明：
// 输入数据涵盖地理、天气、光伏、储能、电价、负荷、环境等所有关键要素。
// 业务处理包括数据采集、能量管理、AI预测与优化（TimesNet+XGBoost）。
// 输出结果包括各类预测、优化建议、经济分析、设备监控和调度方案。
// 目标与价值体现微电网智能化、经济性、灵活性和新能源消纳。

经济型分析包括：峰谷套利、光伏消纳、需求响应、负荷调节、动态增容带来的经济效益统计
防逆流：如果分布式光伏发电量超过本地负荷时，多余电能逆流到主网，会造成主网冲击或违反并网协议。通过监测并网点表计的功率数据，判断是否发生逆流，如果发生逆流，通过EMS或逆变器自动切断分布式电源或自动告警，自动调节分布式电源出力或储能充放电。
能源管理界面：能耗与排放概览、电价配置、能耗与用能成本分析、能流图、能耗和排放报表、能耗KPI管理、微电网单线图
微网优化界面: 发电量预测结果、储能充放策略优化建议、经济性分析结果、微网设备状态


## 2. 系统架构

```mermaid
flowchart BT
    subgraph 上级平台
        %% 调度系统
        subgraph 虚拟电厂/调度平台
            H1[需求响应/负荷调节]
        end

        %% 企业系统
        subgraph 企业管理平台
            H4[资产数据/能源数据/优化数据获取]
        end
    end

    %% 外部平台
    subgraph 外部平台
        H2[天气预报API]
        H3[现货市场价格API]
    end

    %% NXpower Matrix MG
    subgraph NXpower_Matrix_MG
        subgraph 统一风格显示界面
            G5[配电管理界面：预置NXpower AMS站级页面]
            G6[能源管理界面：预置NXpower EMS站级页面]
            G11[NXpower管理界面
            保持不变]
            G7[微网优化界面：融入NXpower EMS站级页面]

            G12[MGMS管理界面
            保持不变]
        end

        subgraph 后端服务
            G1[NXpower_Base服务]
            G2[NXpower软件]
            G3[MGMS软件]
            G10[北向接口服务]
        end

        subgraph 对外接口
            G4[T104接口]
            G9[REST API接口]
        end

        G8[外网连接路径二：NXpower Matrix北向网口接外网]
    end


    %% NXpower Edge网关
    subgraph NXpower_Edge网关
        F1[南向接入: 
        Modbus RTU/TCP]
        F2[数据采集与规约转换]
        F3[外网连接路径一：NXpower Edge 4G扩展]
        F4[网关输出: MQTT]
        F5[南向接入: 
        IEC61850/Modbus]
    end

    %% 南向设备层
    subgraph 南向设备层
        E1[电网: 总表]
        E2[电负荷: 表计]
        E3[光伏系统: 逆变器]
        E4[储能系统: PCS]
        E5[智能配电设备]
    end

    %% 数据链路（自上而下）
    G4 <--T104--> H1
    G9 <--REST API--> H4
    H2 --REST API--> G8
    H3 --REST API--> G8
    G2 --> G11
    G2 --> G6
    G2 --> G5
    G3 --> G12
    G3 --> G7
    G10 <--> G4
    G10 <--> G9
    G1 <--> G10
    G1 <--REST API--> G3
    F3 --> G3
    G8 --> G3
    G1 --> G2
    H2 -.REST API.-> F3
    H3 -.REST API.-> F3
    F4 <--> G1
    F2 <--> F4
    F1 <--> F2
    E1 --> F1
    E2 --> F1
    E3 --> F1
    E4 <--> F1
    E5 <--> F5
    F5 <--> F2
    %% 说明
    classDef product fill:#e6f7ff,stroke:#1890ff,stroke-width:2px;
    class F1,F2,F3,F4,F5 product;
    class G1,G2,G3,G5,G6,G7,G10 product;
    classDef priority1 fill:#d6f5d6,stroke:#52c41a,stroke-width:2px;
    classDef priority2 fill:#fffbe6,stroke:#faad14,stroke-width:2px;
    classDef priority3 fill:#ffe6e6,stroke:#f5222d,stroke-width:2px;
    
    class H2 priority1;
    class H1 priority2;
    class H3 priority3;

```


---


