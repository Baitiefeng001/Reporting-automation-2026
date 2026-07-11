# 企业数据自动化平台架构（V1.0）

> 目标：建立一套稳定、安全、可维护的企业数据自动化体系，实现**开发、生产、通知、AI分析**职责分离。

---

# 一、总体架构

```
                 ┌─────────────────────────────┐
                 │        家里电脑（研发）        │
                 │-----------------------------│
                 │ Cursor                      │
                 │ Claude Code                │
                 │ Codex                      │
                 │ Git / GitHub              │
                 │ 单元测试                   │
                 └──────────────┬──────────────┘
                                │
                        GitHub Release(Tag)
                                │
──────────────────────────部署边界──────────────────────────
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │      公司电脑（生产环境）      │
                 │-----------------------------│
                 │ SQL Server                  │
                 │ Python                      │
                 │ Excel                       │
                 │ Power BI                    │
                 │ Windows Task Scheduler      │
                 └──────────────┬──────────────┘
                                │
                     SharePoint / OneDrive
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │ Hermes（通知 + AI总结）      │
                 │-----------------------------│
                 │ 飞书                         │
                 │ Teams                       │
                 │ 邮件                         │
                 │ AI摘要                       │
                 └─────────────────────────────┘
```

---

# 二、职责划分

## 家里电脑（开发环境）

负责：

* Python开发
* SQL编写
* Cursor辅助开发
* Claude Code重构
* Codex代码Review
* Git版本管理
* 单元测试
* Release发布

禁止：

* 连接生产数据库修改数据
* 保存生产敏感数据
* 人工修改正式报表

---

## 公司电脑（生产环境）

负责：

* SQL取数
* Python ETL
* 数据校验
* Excel生成
* HTML/PDF生成
* Power BI刷新
* Windows定时运行

禁止：

* AI开发
* Git实验代码
* 云服务器保存数据库密码

---

## Hermes（通知中心）

负责：

* 接收运行状态
* AI总结日报
* 飞书推送
* Teams通知
* 邮件发送
* 日志记录

禁止：

* 保存SQL账号密码
* 保存原始订单数据
* 保存客户信息
* 保存工资数据
* 保存合同数据

---

# 三、数据流

每天：

```
08:30

↓

Windows Task Scheduler

↓

run_daily_report.py

↓

SQL查询

↓

Python清洗

↓

数据校验

↓

生成Excel

↓

生成HTML/PDF

↓

生成metrics_summary.json

↓

刷新Power BI

↓

上传SharePoint

↓

通知Hermes

↓

Hermes生成日报

↓

飞书 / Teams / 邮件
```

---

# 四、目录结构

```
reporting/

├── sql/
│   ├── daily_sales.sql
│   ├── customer_alert.sql
│   └── inventory.sql
│
├── src/
│   ├── extract.py
│   ├── transform.py
│   ├── validate.py
│   ├── render_excel.py
│   ├── render_html.py
│   ├── publish.py
│   └── run_daily_report.py
│
├── templates/
│   ├── report.xlsx
│   └── report.html.j2
│
├── config/
│   ├── config.example.yaml
│   └── config.local.yaml
│
├── tests/
│   ├── test_extract.py
│   ├── test_transform.py
│   └── test_validate.py
│
├── output/
│   ├── 2026-07-11/
│   ├── 2026-07-12/
│   └── ...
│
├── history/
│   └── daily_metrics.csv
│
├── logs/
│
└── README.md
```

---

# 五、标准输出

每次运行必须生成：

```
output/

├── report.xlsx
├── report.pdf
├── report.html
├── metrics_summary.json
├── validate_result.json
└── run.log
```

---

# 六、数据校验（Validate）

没有数据校验，就没有自动化。

至少检查：

* 空值
* 重复数据
* 金额对账
* 行数异常
* 环比异常
* 同比异常
* 主键唯一性
* 时间连续性

如果校验失败：

```
停止生成日报

发送报警

等待人工确认
```

---

# 七、AI读取范围

AI只能读取：

```
metrics_summary.json
```

例如：

```json
{
  "date":"2026-07-11",
  "gmv":12860000,
  "margin":31.8,
  "margin_change":-1.7,
  "new_customer":1042,
  "yoy":18.4,
  "mom":6.2,
  "alerts":[
      "华东新客下降18%"
  ],
  "report_link":"SharePoint链接"
}
```

AI禁止读取：

* 原始SQL
* Excel明细
* 客户名单
* SKU明细
* 手机号
* 合同
* 工资
* 订单流水

---

# 八、Git开发流程

```
main
 │
 ├── release/v1.0
 │
 ├── feature/add_margin
 │
 ├── feature/fix_validate
 │
 └── feature/customer_alert
```

开发流程：

```
Feature开发

↓

本地测试

↓

Pull Request

↓

Code Review

↓

Merge Main

↓

Tag

↓

Release

↓

公司电脑部署Release版本
```

生产环境永远部署：

```
Release版本
```

禁止：

```
git pull main
```

---

# 九、Power BI定位

Power BI只负责：

```
读取

↓

展示

↓

交互分析
```

不要：

* ETL
* 数据清洗
* 大量Merge
* 复杂Replace
* 主数据维护

所有数据处理放到Python完成。

---

# 十、AI工具分工

| 工具             | 用途             |
| -------------- | -------------- |
| Cursor         | 日常Python开发     |
| Claude Code    | 重构、大规模修改       |
| Codex          | 第二意见、代码Review  |
| Office Copilot | Excel、PPT、临时问数 |
| Hermes         | 调度、通知、日报总结     |
| Power BI       | 可视化展示          |
| Excel          | 最终交付物          |

---

# 十一、安全原则

允许离开公司电脑的数据：

```json
{
    "status":"success",
    "date":"2026-07-11",
    "gmv_yoy":18.4,
    "margin_change":-1.7,
    "alerts":[
        "华东新客下降"
    ],
    "report_link":"SharePoint"
}
```

禁止离开公司电脑的数据：

* SQL查询结果
* 客户资料
* 门店资料
* SKU销售明细
* 合同
* 工资
* 手机号
* 原始订单
* 原始Excel

---

# 十二、实施路线图

## 第一阶段：本地自动化

* Python连接SQL
* 自动生成Excel
* 自动生成PDF
* 数据校验
* Windows定时任务

目标：

即使没有AI，也能每天自动生成正式日报。

---

## 第二阶段：Git工程化

* Git
* GitHub
* Branch
* Tag
* Release
* 自动部署

目标：

所有代码可追踪、可回滚、可审计。

---

## 第三阶段：AI总结

Hermes读取：

```
metrics_summary.json
```

自动生成：

* 日报摘要
* 管理建议
* 异常提醒

发送：

* 飞书
* Teams
* 邮件

---

## 第四阶段：持续优化

增加：

* 单元测试
* 日志分析
* 自动报警
* 历史趋势分析
* 周报
* 月报
* KPI看板
* AI问数助手

---

# 核心设计原则

1. 开发与生产隔离
2. 数据与AI隔离
3. 校验优先于报告
4. Git是唯一代码来源
5. Release而不是直接部署开发代码
6. AI负责解释数据，而不是创造数据
7. Power BI负责展示，而不是ETL
8. Excel是交付物，而不是数据库
9. 公司电脑是唯一生产数据入口
10. 自动化的目标是稳定、可追溯、可维护，而不是炫技。
