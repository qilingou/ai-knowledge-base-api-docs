# 科研平台私有知识库 API 使用说明

本 API 旨在为 AI 提供对科研论文知识库、数据集知识库、期刊知识库的直接查询能力，支持论文检索、数据集推荐、期刊分区查询、征稿启事查询、代码执行等。

## 基本信息

- **服务名称**: JieclPaperTool（科研平台私有知识库插件）
- **版本**: v1
- **Base URL**: `https://ai1.jiecl.com/api/CozePlugin`
- **字符编码**: `UTF-8`

---

## 1. 核心功能模块

### 📄 论文知识库
- 按主题关联检索期刊论文
- 查询 Arxiv 预印本论文
- 查询论文研究设计（工具变量、稳健性检验）

### 📊 数据集知识库
- 检查数据变量是否存在（推荐数据集）

### 📰 期刊知识库
- 查询期刊分区（中科院/JCR/SSCI）
- 获取期刊征稿启事与热门选题

### 💻 代码执行
- 远程执行 Python 代码
- 远程执行 Stata 代码

---

## 2. 接口一览

| 接口 | 方法 | 功能 |
|------|------|------|
| `/GetPaperRelevanceAndRecommendedPapers` | POST | 主题关联论文检索 |
| `/QueryJournalArticleForArxiv` | POST | Arxiv 预印本论文查询 |
| `/QueryJournalArticleForPaperMethod` | POST | 论文研究设计/工具变量查询 |
| `/QueryJournalCallForPapersData` | POST | 期刊征稿启事查询 |
| `/GetCallForPaperHotTopic` | POST | 期刊热门选题检索 |
| `/GetJournalPartition` | POST | 期刊分区查询 |
| `/GetRecommendedDatasetsForCoze` | POST | 检查数据变量是否存在/推荐数据集 |
| `/GetRecordsBySnapshotIds` | POST | 根据快照ID获取完整查询结果 |
| `/ContainerCodeExecutor` | POST | Python/Stata 代码执行 |

---

## 3. 接口详细说明与示例

### 3.1 论文检索类

#### 📌 主题关联论文检索 `POST /GetPaperRelevanceAndRecommendedPapers`
根据主题查询关联的期刊论文，支持语义搜索和关键词过滤。

```
// 请求 - 搜索"数字经济"与"绿色发展"相关的论文
{
  "Topic": "数字经济对绿色发展的作用机制",
  "KeyWords": ["数字经济", "绿色技术创新", "碳排放"],
  "KeyWordRelation": "and",
  "TopN": 10,
  "Grade": ["CSSCI"],
  "Major": ["经济学"],
  "JournyNames": [],
  "StartDate": "2023-01-01",
  "EndDate": "2025-12-31",
  "SemanticThreshold": "0.5",
  "SearchScope": ["title", "keywords", "descs"],
  "Type": []
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Topic | string | ✅ | 论文主题 |
| KeyWords | string[] | ❌ | 必须出现的关键词（不要把主题也放入） |
| KeyWordRelation | string | ❌ | 关键词关系：`and` 或 `or`，默认 `and` |
| TopN | int | ❌ | 每页数量，默认10 |
| Grade | string[] | ❌ | 期刊级别 |
| Major | string[] | ❌ | 学科领域 |
| JournyNames | string[] | ❌ | 指定期刊 |
| StartDate | string | ❌ | 开始日期 `yyyy-MM-dd` |
| EndDate | string | ❌ | 结束日期 `yyyy-MM-dd` |
| SemanticThreshold | string | ❌ | 语义搜索阈值，默认"0.5" |
| SearchScope | string[] | ❌ | 搜索范围：`title`/`keywords`/`descs` |
| Type | int[] | ❌ | 期刊类型 |

**响应包含：**
- `RelatedPapers`: 关联论文列表（标题、摘要、关键词、期刊、URL、语义得分）
- `TotalCount`: 总论文数

---

#### 📌 Arxiv 预印本论文查询 `POST /QueryJournalArticleForArxiv`
查询 Arxiv 预印本论文。

```
// 请求
{
  "Topic": "carbon emissions and economic growth",
  "TopN": 10
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Topic | string | ✅ | 搜索内容 |
| TopN | int | ❌ | 每页数量，默认10 |
| Filter | string | ❌ | 过滤条件 |

**响应额外字段：** `paper_id`, `authors`, `abstract`, `pdf_url`, `subjects`, `published`

---

#### 📌 论文研究设计查询 `POST /QueryJournalArticleForPaperMethod`
查询论文中的研究设计、工具变量、稳健性检验等内容。

```
// 请求
{
  "Topic": "双重差分法的工具变量选择",
  "TopN": 10
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Topic | string | ✅ | 搜索内容 |
| TopN | int | ❌ | 每页数量，默认10 |
| Filter | string | ❌ | 过滤条件 |

**响应额外字段：** `researchMethodAbstract`, `IVSelectionReasons`, `robustnessCheck`

---

### 3.2 期刊信息类

#### 📌 期刊分区查询 `POST /GetJournalPartition`
根据期刊名称查询中科院分区和 JCR/SSCI 分区信息。

```
// 请求
{
  "JournalName": ["Academic Pathology", "中国工业经济"]
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| JournalName | string[] | ✅ | 期刊名称数组 |

**响应包含：**
- `ZKYPartition`: 中科院分区（1-4）
- `SSCIPartition`: SSCI分区
- `Category`: 大类学科
- `Top`: 是否顶刊
- `OpenAccess`: 是否开放获取
- `PublisherName`: 出版社
- `ISSN` / `EISSN`: ISSN号

---

#### 📌 期刊热门选题检索 `POST /GetCallForPaperHotTopic`
检索期刊中的热门选题和征稿启事。

```
// 请求
{
  "HotTopics": "数字经济",
  "CallForPaperType": "期刊",
  "JournalCount": 10,
  "Grade": ["CSSCI"],
  "Major": [],
  "JournyNames": [],
  "Type": [2],
  "BeginTime": "",
  "EndTime": ""
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| HotTopics | string | ❌ | 热点选题关键词 |
| CallForPaperType | string | ❌ | 征稿分类：`期刊` 或 `会议`，默认`期刊` |
| JournalCount | int | ❌ | 期刊数量，默认10 |
| Grade | string[] | ❌ | 期刊级别 |
| Major | string[] | ❌ | 学科领域 |
| JournyNames | string[] | ❌ | 指定期刊 |
| Type | int[] | ❌ | 期刊类型：2=有征稿启事 |
| BeginTime | string | ❌ | 开始日期 `yyyy-MM-dd` |
| EndTime | string | ❌ | 结束日期 `yyyy-MM-dd` |

---

#### 📌 期刊征稿启事查询 `POST /QueryJournalCallForPapersData`
语义查询期刊征稿启事。

```
// 请求
{
  "Topic": "人工智能",
  "Filter": ""
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Topic | string | ❌ | 语义检索内容 |
| Filter | string | ❌ | 过滤条件 |

---

### 3.3 数据集类

#### 📌 检查数据变量是否存在/推荐数据集 `POST /GetRecommendedDatasetsForCoze`

```
// 请求
{
  "req": [
    {
      "Id": "1",
      "IndicatorName": "新型基础设施水平",
      "Topic": "新型基础设施对出口韧性的影响机制研究"
    },
    {
      "Id": "2",
      "IndicatorName": "出口韧性",
      "Topic": "新型基础设施对出口韧性的影响机制研究"
    }
  ],
  "SemanticThreshold": "0.35"
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| req | array | ✅ | 待查询的变量数组 |
| req[].Id | string | ✅ | 变量序号 |
| req[].IndicatorName | string | ✅ | 数据变量名称 |
| req[].Topic | string | ✅ | 研究主题 |
| SemanticThreshold | string | ❌ | 语义搜索阈值，默认"0.35" |

**响应包含：**
- `IsSuccess`: 是否存在
- `SnapshotId`: 匹配到的搜索快照ID（可后续用 `/GetRecordsBySnapshotIds` 获取完整数据）

---

#### 📌 根据快照ID获取完整查询结果 `POST /GetRecordsBySnapshotIds`
> ⚠️ **核心接口** — 前面的查询接口会先将结果存入快照表（轻量返回），再通过此接口根据快照ID拉取完整数据，大幅降低数据传输量。

根据快照ID获取真正的结果数据，根据不同业务类型返回不同结构：
- 快照ID对应期刊论文 → 返回论文数据
- 快照ID对应数据集 → 返回数据集信息
- 快照ID对应征稿启事 → 返回征稿数据
- 以此类推，按 `SearchType` 字段区分

```
// 请求 - 根据快照ID获取完整结果
{
  "SnapshotIds": [2029856011514417152]
}

// 请求 - 批量获取多个快照结果
{
  "SnapshotIds": [2029856011514417152, 2035261418055536640]
}
```
> 注意：该接口与其他接口共享相同的 Base URL：`https://ai1.jiecl.com/api/CozePlugin/GetRecordsBySnapshotIds`

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| SnapshotIds | int64[] | ✅ | 快照ID数组（来自其他接口返回的 `SnapshotId`） |

**响应结构：**

```json
{
  "IsSuccess": true,
  "Message": "",
  "Data": [
    {
      "SearchType": "期刊论文",
      "Records": [
        // 根据 SearchType 不同，Records 的结构不同
        // 期刊论文：title, keywords, descs, source, url, journallevel, issue ...
        // 数据集：name, indicator, url, year_start, year_end ...
      ]
    }
  ]
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| IsSuccess | boolean | 请求是否成功 |
| Message | string | 错误信息（失败时） |
| Data | array | 快照结果数组 |
| Data[].SearchType | string | 业务类型（期刊论文/数据集/征稿启事等） |
| Data[].Records | array | 该快照对应的完整数据记录 |

---

### 3.4 代码执行

#### 📌 Python/Stata 代码执行 `POST /ContainerCodeExecutor`
在远程容器中执行 Python 或 Stata 代码。

```
// 请求 - 执行 Python 代码
{
  "Code": "import pandas as pd\ndf = pd.read_csv('data.csv')\nprint(df.describe())",
  "ContainerIp": "192.168.61.99"
}

// 请求 - 执行 Stata 代码
{
  "Code": "stata.run('help regress')",
  "ContainerIp": "192.168.61.99"
}
```

**请求参数说明：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Code | string | ✅ | Python 或 Stata 代码 |
| ContainerIp | string | ✅ | 容器IP，默认 `192.168.61.99` |

**响应包含：**
- `Result`: 代码执行输出结果（字符串）
- `FileBytes`: 输出文件的 Base64 编码（如有）
- `IsSuccess`: 执行是否成功

---

## 4. 推荐使用流程

### 论文研究流程
1. **选题探索** → 调用 `/GetCallForPaperHotTopic` 获取期刊热门选题
2. **文献检索** → 调用 `/GetPaperRelevanceAndRecommendedPapers` 语义检索关联论文，获取 `SnapshotId`
3. **获取完整数据** → 调用 `/GetRecordsBySnapshotIds` 根据 `SnapshotId` 拉取完整论文数据
4. **方法学习** → 调用 `/QueryJournalArticleForPaperMethod` 学习研究设计
5. **期刊选择** → 调用 `/GetJournalPartition` 查看期刊分区

### 数据驱动研究流程
1. **变量确认** → 调用 `/GetRecommendedDatasetsForCoze` 检查数据变量是否存在，获取 `SnapshotId`
2. **获取完整数据** → 调用 `/GetRecordsBySnapshotIds` 根据 `SnapshotId` 拉取完整数据集信息
3. **代码分析** → 调用 `/ContainerCodeExecutor` 执行分析代码

### 典型复合场景示例

**场景：研究"新质生产力对碳排放的影响"**

```
// 第1步：检查数据变量
POST /GetRecommendedDatasetsForCoze
{
  "req": [
    {"Id": "1", "IndicatorName": "新质生产力", "Topic": "新质生产力对碳排放的影响"},
    {"Id": "2", "IndicatorName": "碳排放", "Topic": "新质生产力对碳排放的影响"}
  ]
}

// 第2步：检索相关论文（获取快照ID）
POST /GetPaperRelevanceAndRecommendedPapers
{
  "Topic": "新质生产力对碳排放的影响",
  "KeyWords": ["新质生产力", "碳排放"],
  "KeyWordRelation": "and",
  "TopN": 20
}
// 响应中包含 SnapshotId，例如 "2035261418055536640"

// 第3步：根据快照ID拉取完整论文数据
POST /GetRecordsBySnapshotIds
{
  "SnapshotIds": [2035261418055536640]
}

// 第4步：查看研究方法
POST /QueryJournalArticleForPaperMethod
{
  "Topic": "新质生产力 碳排放 双重差分"
}

// 第5步：执行分析代码
{
  "Code": "import pandas as pd\n# ... 分析代码 ...",
  "ContainerIp": "192.168.61.99"
}
```

---

## 5. 通用过滤参数说明

多个接口共享以下过滤参数：

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `Grade` | 期刊级别 | `["CSSCI"]`, `["UTD"]`, `["权威A"]`, `["CSSCI","北大核心"]` |
| `Major` | 学科领域 | `["经济学"]`, `["管理学"]`, `["教育学"]` |
| `JournyNames` | 指定期刊 | `["经济研究"]`, `["中国工业经济"]` |
| `Type` | 期刊类型 | `[1]`=提供代码数据, `[2]`=有征稿启事 |
| `BeginTime` / `StartDate` | 开始日期 | `"2023-01-01"` |
| `EndTime` / `EndDate` | 结束日期 | `"2025-12-31"` |
| `TopN` / `TakeNum` | 返回数量 | `10` |

---

> 📝 **使用提示**：本 API 可供任意 AI 通过对话调用。只需将本文档提供给 AI，AI 即可根据用户的研究需求，自动组合调用各接口完成论文检索、数据分析、代码执行等复杂任务。
