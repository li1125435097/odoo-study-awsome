在 **Odoo 17** 中，`<dashboard>` 视图（也称为 **仪表盘视图**）是一种特殊的复合视图，用于在一个页面中**组合多个子视图**（如 `graph`、`list`、`kanban`、`pivot` 等），实现数据概览和分析。它常用于 **首页看板、管理驾驶舱、业务监控面板** 等场景。

> ⚠️ **重要前提**：  
> **`<dashboard>` 视图仅在 Odoo Enterprise（企业版）中可用**。社区版（Community Edition）不支持此功能。如果你使用的是社区版，该视图将无法加载或报错。

---

## ✅ 综合示例：包含所有可用属性的 Dashboard 表单（Odoo 17）

```xml
<!-- 文件：views/comprehensive_dashboard_view.xml -->
<odoo>
    <record id="view_comprehensive_dashboard" model="ir.ui.view">
        <field name="name">comprehensive.dashboard</field>
        <field name="model">your.model</field> <!-- 注意：dashboard 通常绑定一个主模型，但子视图可来自不同模型 -->
        <field name="arch" type="xml">
            <!-- 
                dashboard 根元素属性说明：
                - js_class: 指定自定义 JS 控制器（高级用法）
                - class: 添加 CSS 类到整个仪表盘容器
                - sample: 是否启用示例数据模式（开发调试用）
                - create: 是否显示“创建”按钮（通常设为 false，因 dashboard 是只读汇总）
                - limit: 对主模型记录的限制（但子视图通常独立查询）
            -->
            <dashboard
                js_class="custom_dashboard_controller"
                class="o_comprehensive_dashboard"
                sample="False"
                create="False"
                limit="0"
            >
                <!-- 
                    子视图通过 <view> 标签嵌入
                    每个 <view> 定义一个独立的子组件（如图表、列表等）
                -->

                <!-- 子视图 1：销售趋势图（graph） -->
                <view
                    type="graph"
                    model="sale.order"
                    string="Sales Trend"
                    help="Monthly sales performance"
                >
                    <graph type="bar">
                        <field name="date_order" type="row"/>
                        <field name="amount_total" type="measure"/>
                    </graph>
                </view>

                <!-- 子视图 2：待办任务列表（list） -->
                <view
                    type="list"
                    model="project.task"
                    string="Pending Tasks"
                    help="Tasks requiring your attention"
                >
                    <list default_order="priority desc">
                        <field name="name"/>
                        <field name="user_ids" widget="many2many_tags"/>
                        <field name="deadline"/>
                    </list>
                </view>

                <!-- 子视图 3：机会看板（kanban） -->
                <view
                    type="kanban"
                    model="crm.lead"
                    string="Opportunities"
                    help="Sales pipeline overview"
                >
                    <kanban default_group_by="stage_id">
                        <field name="name"/>
                        <field name="partner_id"/>
                        <field name="expected_revenue"/>
                        <field name="stage_id"/>
                        <templates>
                            <t t-name="kanban-box">
                                <div class="oe_kanban_card oe_kanban_global_click">
                                    <strong><t t-esc="record.name.value"/></strong>
                                    <div>Revenue: <t t-esc="record.expected_revenue.value"/></div>
                                </div>
                            </t>
                        </templates>
                    </kanban>
                </view>

                <!-- 子视图 4：利润透视表（pivot） -->
                <view
                    type="pivot"
                    model="account.move.line"
                    string="Profit Analysis"
                    help="Breakdown by product and region"
                >
                    <pivot>
                        <field name="product_id" type="row"/>
                        <field name="partner_id" type="col"/>
                        <field name="balance" type="measure"/>
                    </pivot>
                </view>

                <!-- 
                    注意：
                    - 每个 <view> 可以指定不同的 model
                    - 子视图的 XML 结构必须完整（如 <graph> 内部需有字段定义）
                    - string: 子视图标题（显示在卡片顶部）
                    - help: 悬停提示或描述（部分版本显示在标题下方）
                -->

            </dashboard>
        </field>
    </record>
</odoo>
```

> 💡 **关键说明**：
> - **每个 `<view>` 是一个独立的子视图组件**，可来自不同模型。
> - **`type` 必须是 Odoo 支持的视图类型**：`graph`, `list`, `kanban`, `pivot`, `calendar` 等。
> - **`string` 是子视图的显示标题**，`help` 提供辅助说明。
> - **主 `<dashboard>` 的 `model` 字段通常无实际作用**（子视图通过各自的 `model` 属性指定数据源）。
> - **布局由 Odoo 自动管理**（响应式网格，用户可拖拽调整位置和大小）。

---

## 📚 Dashboard 视图所有可操作属性总结

### 一、`<dashboard>` 根元素属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `js_class` | string | — | 指定自定义 JS 控制器（继承 `DashboardController`） |
| `class` | string | — | 添加 CSS 类到仪表盘容器 |
| `sample` | boolean | `False` | 启用示例数据（开发调试用） |
| `create` | boolean | `True` | 是否显示“创建”按钮（通常设为 `False`） |
| `limit` | integer | — | 对主模型记录的限制（**通常无效**，因子视图独立查询） |

> ⚠️ 注意：
> - `default_order`、`editable`、`delete` 等属性 **不适用于 dashboard**。
> - **社区版不支持**：此视图仅在 Odoo Enterprise 中有效。

---

### 二、`<view>` 子元素属性（核心）

每个 `<view>` 定义一个子视图，其属性如下：

| 属性 | 必填 | 说明 |
|------|------|------|
| `type` | ✅ 是 | 子视图类型（`"graph"`, `"list"`, `"kanban"`, `"pivot"`, `"calendar"`） |
| `model` | ✅ 是 | 子视图的数据模型（如 `"sale.order"`） |
| `string` | ❌ 否 | 子视图标题（显示在卡片顶部） |
| `help` | ❌ 否 | 辅助说明文本（hover 或显示在标题下方） |
| `class` | ❌ 否 | 添加 CSS 类到子视图容器 |

> 🔍 **内部结构**：
> - `<view>` 内部必须包含对应类型的完整视图定义（如 `<graph>...</graph>`）。
> - 子视图的属性（如 `graph` 的 `type="bar"`）按各自视图规则配置。

---

### 三、用户交互功能（自动提供）

| 功能 | 说明 |
|------|------|
| **拖拽布局** | 用户可拖动子视图调整位置 |
| **调整大小** | 用户可拉伸子视图改变尺寸 |
| **刷新数据** | 每个子视图可独立刷新 |
| **导出/打印** | 支持导出子视图为 Excel/PDF（企业版功能） |
| **保存布局** | 用户自定义的布局会自动保存到个人设置 |

---

### 四、限制与注意事项

1. **仅限企业版**：社区版无法使用 `<dashboard>` 视图。
2. **性能影响**：每个子视图独立执行 RPC 查询，过多子视图可能导致加载缓慢。
3. **权限控制**：子视图受各自模型的访问权限控制（若用户无权访问某模型，则该子视图空白或报错）。
4. **无全局筛选**：各子视图的搜索条件相互独立（无法像普通视图那样共享 SearchView）。
5. **不支持 form 视图**：不能将 `<form>` 作为子视图嵌入（因 form 是单记录视图）。

---

### 五、最佳实践建议

- **聚焦关键指标**：每个子视图应展示一个明确的业务洞察。
- **合理选择视图类型**：
  - 趋势 → `graph`
  - 明细列表 → `list`
  - 流程状态 → `kanban`
  - 多维分析 → `pivot`
- **提供清晰标题**：使用 `string` 和 `help` 帮助用户理解数据含义。
- **测试权限**：确保目标用户组有权限访问所有子视图的模型。
- **避免过度复杂**：建议子视图数量 ≤ 6，保持界面清爽。

---

## 🖥️ 典型应用场景

```xml
<!-- 销售经理仪表盘 -->
<dashboard>
    <view type="graph" model="sale.order" string="Monthly Revenue">
        <graph type="line">...</graph>
    </view>
    <view type="kanban" model="crm.lead" string="Pipeline">
        <kanban default_group_by="stage_id">...</kanban>
    </view>
    <view type="list" model="sale.order" string="Recent Orders">
        <list>...</list>
    </view>
</dashboard>
```

---

## 🔧 替代方案（社区版用户）

如果你使用 **Odoo 社区版**，可通过以下方式模拟仪表盘：

1. **自定义 OWL 组件**：
   - 创建一个 `Dashboard` OWL 组件。
   - 使用 `useService("orm")` 手动调用多个模型的聚合方法。
   - 在模板中组合 `graph`、`list` 等子组件。

2. **使用网页构建器（Website Builder）**：
   - 创建一个 `/dashboard` 页面。
   - 通过 Snippet 嵌入多个报表（需自定义后端接口）。

3. **第三方模块**：
   - 安装社区模块如 `web_widget_multi_chart` 或 `board`（旧版看板，Odoo 15+ 已弃用）。

---

通过以上配置，你可以在 **Odoo Enterprise 17** 中构建强大的业务仪表盘。如需动态数据联动或高级交互，建议结合 **自定义 JS 控制器** 和 **RPC 方法** 实现。