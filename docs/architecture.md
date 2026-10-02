# XYZ 架构说明

> 本图是面向读者的结构示意图，不是运行时调用追踪；具体实现会随源代码快照继续迭代。

```mermaid
flowchart LR
    U[用户<br/>User] --> UI[SwiftUI 界面<br/>Conversation · Projects · Writing]
    UI --> STATE[AppState 与本地 Stores<br/>草稿 · 会话 · 队列 · 成果]

    STATE --> ROUTER{模型路由<br/>Model router}
    ROUTER --> TEXT[XYZ 3.8<br/>本地对话与图像理解<br/>off · low · medium · xhigh]
    ROUTER --> IMAGE[XYZ Image<br/>本地图片生成]

    IMAGE --> QUEUE[Image queue<br/>暂停 · 继续 · 取消 · 重试]
    QUEUE --> ENGINE[专用图片引擎<br/>按需启动 · 连续任务复用 · 空闲回收]

    UI --> MATERIALS[附件与材料<br/>图片 · PDF · Word · TXT · Markdown]
    MATERIALS --> KNOWLEDGE[知识库与证据阅读<br/>本地索引 · 定位原文]
    KNOWLEDGE --> WRITING[写作工作室与成果库<br/>版本 · 复核 · 导出]

    STATE --> STORAGE[(本机工作区<br/>会话 · 项目 · 草稿 · 记录)]

    UI --> MODE{安全 / 联网模式}
    MODE -->|安全| LOCAL[本地处理<br/>不发起公开检索]
    MODE -->|联网| WEB[公开网络能力<br/>仅发送当前检索词或网址]

    classDef surface fill:#0c1714,stroke:#55d99a,color:#eafff4;
    classDef core fill:#12231d,stroke:#86f0bd,color:#f1fff7;
    classDef boundary fill:#20231f,stroke:#d1b96f,color:#fff8df;
    class U,UI surface;
    class STATE,ROUTER,TEXT,IMAGE,QUEUE,ENGINE,MATERIALS,KNOWLEDGE,WRITING,STORAGE core;
    class MODE,LOCAL,WEB boundary;
```

## 读图方式

### 界面与状态

界面把输入、模型选择、项目、写作和图片任务交给共享的应用状态层。会话、草稿、队列、项目和成果各自有持久化模型，因而图片任务可以在后台运行，用户仍能浏览历史或切换到另一个对话。

### 两条模型路径

`XYZ 3.8` 负责本地文本对话和图像理解，并提供四档思考强度。`XYZ Image` 走独立的图片任务队列和专用引擎，队列结束或空闲后释放资源。两条路径共享工作区的交互原则，但不强行共用同一套资源生命周期。

### 材料、证据与写作

附件先进入本地材料处理，再按需要进入知识库、证据阅读或写作工作室。证据卡片保留文件、页码或段落等定位信息，写作成果保留版本和来源回链，便于从成稿回到材料。

### 网络边界

安全模式把工作限制在本机；联网模式只为用户主动发起的公开检索或网址阅读提供网络能力。模型、历史对话和本地附件不因为打开普通对话而自动上传。

### 当前边界

模型权重、应用源码和用户资料不属于公开展示材料。图与说明用于解释设计；它们不覆盖所有运行时细节，也不将本机安装版本等同于私有远端源码版本。
