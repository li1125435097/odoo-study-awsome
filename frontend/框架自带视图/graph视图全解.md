在 **Odoo 17** 中，`<graph>` 视图（也称图表视图）用于以可视化方式（柱状图、折线图、饼图等）展示数据聚合结果。它基于模型字段进行分组（`group by`）和度量（`measure`），常用于报表分析。

与 `<form>`、`<list>`、`<kanban>` 不同，`<graph>` 视图**不直接显示记录列表**，而是对记录集进行 **自动聚合（aggregation）**，因此其结构更简单，但属性高度专注于 **维度（dimensions）** 和 **指标（measures）** 的定义。

---

## ✅ 综合示例：包含所有可用属性的 Graph 表单（Odoo 17）

```xml
<!-- 文件：views/comprehensive_graph_view.xml -->
<odoo>
    <record id="view_comprehensive_graph" model="ir.ui.view">
        <field name="name">comprehensive.graph</field>
        <field name="model">your.model</field>
        <field name="arch" type="xml">
            <!-- 
                graph 根元素属性说明：
                - type: 图表类型（"bar" / "line" / "pie" / "pivot"）
                - stacked: 是否堆叠（仅 bar/line 有效）
                - display_quantity: 是否显示数值标签（部分图表支持）
                - js_class: 自定义 JS 控制器（高级用法）
                - class: 添加 CSS 类
                - sample: 是否启用示例数据模式（开发用）
            -->
            <graph 
                type="bar" 
                stacked="False"
                display_quantity="True"
                js_class="custom_graph_controller"
                class="o_comprehensive_graph"
                sample="False"
            >
                <!-- 
                    字段声明规则：
                    - 第一个 <field> 是 X 轴（维度，dimension）
                    - 后续 <field> 是 Y 轴（指标，measure）
                    - 所有字段必须支持聚合（numeric 字段或带 group_operator 的字段）
                -->

                <!-- X 轴：按此字段分组（维度） -->
                <!-- invisible="1" 通常不需要，因为会显示为标签 -->
                <field name="category_id" type="row"/> <!-- type="row" 是默认值 -->

                <!-- Y 轴：度量字段（指标） -->
                <!-- sum/avg/min/max 由字段的 group_operator 决定（默认 sum） -->
                <field name="amount_total" type="measure"/>

                <!-- 可选：第二个维度（用于多系列图表，如分组柱状图） -->
                <!-- 在 bar/line 图中，这会生成多个系列 -->
                <field name="user_id" type="col"/>

                <!-- 其他可聚合字段（可选） -->
                <field name="quantity" type="measure"/>
                <field name="priority" type="measure"/> <!-- 若 priority 是 integer -->

                <!-- 
                    注意：
                    - 不支持 widget、options、readonly 等属性
                    - 不支持 button、separator 等元素
                    - 所有字段必须能被聚合（即 numeric 或定义了 group_operator）
                -->

            </graph>
        </field>
    </record>
</odoo>
```

> 💡 **关键说明**：
> - **第一个 `<field>`** 是主分组维度（X 轴）。
> - **后续 `<field type="measure">`** 是度量指标（Y 轴）。
> - **`<field type="col">`** 作为第二维度时，会在 bar/line 图中创建多系列（如不同用户的销售额对比）。
> - 字段必须是 **可聚合的**（如 `Integer`, `Float`, `Monetary`），或通过 `_group_by_full` / `group_operator` 支持聚合。
> - `type="row"` 和 `type="col"` 是 Odoo 内部术语，分别表示“行分组”和“列分组”。

---

## 📚 Graph 视图所有可操作属性总结

### 一、`<graph>` 根元素属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `type` | `"bar"` / `"line"` / `"pie"` / `"pivot"` | `"bar"` | 图表类型 |
| `stacked` | boolean | `False` | 是否堆叠（仅 `bar` 和 `line` 有效） |
| `display_quantity` | boolean | `False` | 是否在图表上显示具体数值（如柱顶数字） |
| `js_class` | string | — | 指定自定义 JS 控制器（继承 `GraphController`） |
| `class` | string | — | 添加 CSS 类到图表容器 |
| `sample` | boolean | `False` | 启用示例数据（开发调试用，不加载真实数据） |

> ⚠️ 注意：
> - `create`、`edit`、`delete` 等属性 **不适用于 graph 视图**（它是只读报表）。
> - `default_order`、`limit` 等也无效。

---

### 二、`<field>` 元素属性（在 `<graph>` 中）

| 属性 | 必填 | 说明 |
|------|------|------|
| `name` | ✅ 是 | 模型字段名（必须可聚合） |
| `type` | 否 | `"row"`（默认）、`"col"`、`"measure"` |
| `invisible` | 否 | **无效**（graph 中忽略） |
| 其他属性（`widget`, `options`, `string` 等） | — | **全部无效** |

#### `type` 值详解：

| `type` | 作用 | 示例 |
|--------|------|------|
| `"row"` | 主分组维度（X 轴） | 按 `category_id` 分组 |
| `"col"` | 次分组维度（多系列） | 按 `user_id` 拆分为多个柱子 |
| `"measure"` | 度量指标（Y 轴） | `amount_total` 的总和 |

> 🔍 **内部逻辑**：
> - Odoo 会自动对 `type="measure"` 的字段执行聚合（如 `SUM(amount_total)`）。
> - 聚合函数由字段的 `group_operator` 决定（默认为 `sum`，可设为 `avg`, `min`, `max`, `count`）。

#### 如何自定义聚合函数？
在模型中定义字段时指定：

```python
# models/your_model.py
amount_avg = fields.Float(
    string="Average Amount",
    group_operator='avg'  # 关键：指定聚合方式
)
```

然后在 graph 中使用：
```xml
<field name="amount_avg" type="measure"/>
```

---

### 三、支持的字段类型（可聚合）

| 字段类型 | 是否支持 | 说明 |
|----------|--------|------|
| `Integer` | ✅ | 默认 `sum` |
| `Float` | ✅ | 默认 `sum` |
| `Monetary` | ✅ | 默认 `sum` |
| `Boolean` | ⚠️ 有限 | 可计数（`count`），但需自定义 `group_operator` |
| `Char` / `Text` | ❌ | 不能作为 `measure`（但可作为 `row`/`col` 分组） |
| `Many2one` | ✅（仅作分组） | 可作为 `row`/`col`，显示为关联记录名称 |
| `Selection` | ✅（仅作分组） | 按选项值分组 |
| `Date` / `Datetime` | ✅（仅作分组） | 自动按年/月/日分组（Odoo 智能处理） |

> ✅ **最佳实践**：将 `Date` 字段作为 `row`，Odoo 会自动提供“按年”、“按月”等分组选项。

---

### 四、用户交互功能（自动提供）

- **切换图表类型**：用户可在界面右上角切换 bar/line/pie。
- **调整分组维度**：点击 X 轴标签可下钻（drill down）。
- **导出数据**：支持导出为 Excel/PDF（需企业版或自定义）。
- **筛选**：受当前搜索条件（SearchView）影响。

---

### 五、限制与注意事项

1. **无自定义模板**：不能像 kanban 那样写 QWeb 模板。
2. **无按钮/操作**：纯展示视图，不支持交互按钮。
3. **性能敏感**：大数据集聚合可能较慢，建议配合 domain 过滤。
4. **货币字段**：需确保 `currency_field` 正确设置，否则金额可能显示错误。

---

## ✅ 最佳实践建议

- **主维度放第一**：`<field name="date" type="row"/>` 作为时间轴。
- **合理选择指标**：避免过多 `measure` 字段导致图表混乱。
- **利用 Date 智能分组**：Odoo 对日期字段自动提供多级分组（年→月→日）。
- **命名清晰**：在模型中为聚合字段设置明确的 `string`（如 “Total Sales”）。
- **测试聚合逻辑**：确保 `group_operator` 符合业务需求（如平均单价用 `avg`）。

---

## 📊 示例场景

```xml
<!-- 销售分析：按月份（X轴），显示总销售额和订单数量 -->
<graph type="bar">
    <field name="order_date" type="row"/>     <!-- Odoo 自动按月分组 -->
    <field name="amount_total" type="measure"/>
    <field name="id" type="measure"/>         <!-- count 订单数（id 字段默认 count） -->
</graph>
```

> 💡 技巧：使用 `id` 字段作为 `measure` 可实现 **计数（COUNT）**。

---

通过以上配置，你可以在 Odoo 17 中构建清晰、高效的分析图表。如需更复杂可视化（如地图、甘特图），建议结合 **Dashboard（企业版）** 或自定义 **OWL 组件 + RPC 调用** 实现。