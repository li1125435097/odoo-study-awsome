在 **Odoo 17** 中，`<pivot>` 视图（透视表视图）是一种用于**多维数据聚合与交叉分析**的只读报表视图。它允许用户通过拖放字段到“行”、“列”和“度量值”区域，动态生成类似 Excel 透视表的汇总表格。

虽然 `<pivot>` 的结构看似简单，但它支持一组明确的属性和字段配置方式。以下是对其所有可操作属性的完整说明、一个包含全部可用特性的 XML 示例，以及属性总结。

---

## ✅ 完整示例：包含所有可用属性的 Pivot 视图 XML（带详细注释）

```xml
<!-- 文件：views/sale_report_pivot.xml -->
<odoo>
    <record id="view_sale_report_pivot" model="ir.ui.view">
        <field name="name">sale.report.pivot</field>
        <field name="model">sale.report</field> <!-- 通常为聚合模型或数据库视图 -->
        <field name="arch" type="xml">
            <!-- 
                <pivot> 根元素支持以下 4 个属性：
                - display_quantity: 是否显示具体数值（默认 True）
                - js_class: 指定自定义 JS 控制器（高级用法）
                - class: 添加 CSS 类到容器
                - sample: 启用示例数据模式（仅开发调试）
            -->
            <pivot
                display_quantity="True"
                js_class="custom_pivot_controller"
                class="o_sale_report_pivot"
                sample="False"
            >
                <!-- 
                    字段声明规则：
                    - 所有参与分组或聚合的字段必须在此列出
                    - 即使 invisible="1"，只要用于逻辑也需声明
                    - 字段角色由 type 属性或顺序决定
                -->

                <!-- 行维度（Row）：垂直分组 -->
                <!-- type="row" 可省略（第一个字段默认为 row） -->
                <field name="product_category_id" type="row"/>

                <!-- 列维度（Column）：水平分组 -->
                <!-- type="col" 可省略（第二个字段默认为 col） -->
                <field name="order_date" type="col"/> <!-- Odoo 自动提供年/月/日下钻 -->

                <!-- 度量值（Measure）：聚合指标 -->
                <!-- type="measure" 可省略（第三个及之后字段默认为 measure） -->
                <field name="quantity" type="measure"/>
                <field name="price_subtotal" type="measure"/>
                
                <!-- 使用 id 字段实现 COUNT 聚合 -->
                <field name="id" type="measure"/>

                <!-- 辅助字段（用于 decoration 或 domain，即使不显示也需声明） -->
                <field name="state" invisible="1"/>
                <field name="user_id" invisible="1"/>

                <!-- 
                    动态样式（Decoration）
                    基于字段值添加 CSS 类（如 text-danger）
                    注意：在 pivot 中，decoration 作用于单元格或标题
                -->
                <attribute name="decoration-danger">state == 'cancel'</attribute>
                <attribute name="decoration-warning">price_subtotal &lt; 0</attribute>
                <attribute name="decoration-bf">quantity &gt; 100</attribute>

            </pivot>
        </field>
    </record>
</odoo>
```

> 💡 **关键说明**：
> - **字段角色分配**：
>   - 第1个字段 → `row`
>   - 第2个字段 → `col`
>   - 第3+字段 → `measure`
>   - 显式指定 `type` 可覆盖默认行为。
> - **`display_quantity="True"`**：确保单元格显示数字（而非仅颜色）。
> - **`order_date` 作为 `col`**：Odoo 自动提供时间层级下钻（年 → 季度 → 月）。
> - **`id` 作为 `measure`**：自动执行 `COUNT(*)`。
> - **`decoration-*`**：高亮异常数据（如负利润、已取消订单）。

---

## 📚 Pivot 视图所有可操作属性总结

### 一、`<pivot>` 根元素属性（共 4 个）

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `display_quantity` | boolean | `True` | 是否在单元格中显示具体数值；若为 `False`，仅通过背景色深浅表示大小 |
| `js_class` | string | — | 指定自定义 JavaScript 控制器类名（需继承 `@web/views/pivot/pivot_controller.js`） |
| `class` | string | — | 添加自定义 CSS 类到透视表最外层容器 |
| `sample` | boolean | `False` | 启用示例数据模式（不查询真实数据，仅用于 UI 开发调试） |

> ⚠️ **不支持的属性**（常见误区）：
> - `create`, `edit`, `delete`, `editable`, `default_order`, `limit`, `string`, `domain` 等 —— **全部无效**，因为 pivot 是只读聚合视图。

---

### 二、`<field>` 元素属性（在 `<pivot>` 内部）

| 属性 | 必填 | 有效值 | 说明 |
|------|------|--------|------|
| `name` | ✅ 是 | 字段名 | 模型中的字段名称（必须存在） |
| `type` | ❌ 否 | `"row"` / `"col"` / `"measure"` | 显式指定字段角色（可省略，按顺序自动分配） |
| `invisible` | ❌ 否 | `"1"` | **技术上可写，但无渲染效果**（仅用于确保字段被加载） |

> 🔍 **字段角色自动分配规则**：
> 1. 第一个 `<field>` → `type="row"`
> 2. 第二个 `<field>` → `type="col"`
> 3. 第三个及之后 → `type="measure"`

> ✅ **最佳实践**：显式写出 `type` 提高可读性和维护性。

---

### 三、`<attribute>` 元素：动态样式（Decoration）

| 属性名 | 效果 | 说明 |
|--------|------|------|
| `decoration-danger` | 红色文字/背景 | 如：`state == 'cancel'` |
| `decoration-warning` | 橙色 | 如：`profit < 0` |
| `decoration-success` | 绿色 | 如：`status == 'done'` |
| `decoration-info` | 蓝色 | |
| `decoration-muted` | 灰色 | 如：`active == False` |
| `decoration-bf` | **粗体** | |
| `decoration-it` | *斜体* | |

> 📌 **注意**：
> - 表达式使用 **Python 风格语法**（`==`, `!=`, `<`, `>`, `and`, `or`, `not`）。
> - 所涉及字段必须已在 `<pivot>` 中声明（包括 `invisible="1"` 的字段）。
> - 样式应用范围：**单元格、行标题、列标题**（取决于上下文）。

---

### 四、支持的字段类型

#### 1. **作为维度（`row` / `col`）**
| 字段类型 | 支持 | 说明 |
|----------|------|------|
| `Many2one` | ✅ | 显示关联记录的 `display_name` |
| `Selection` | ✅ | 按选项值分组 |
| `Date` / `Datetime` | ✅ | **智能时间分组**（年 → 季度 → 月 → 日） |
| `Char` / `Text` | ✅（不推荐） | 按文本值分组（高基数性能差） |
| `Boolean` | ✅ | 分为 `True` / `False` 两组 |

#### 2. **作为度量（`measure`）**
| 字段类型 | 聚合方式 | 说明 |
|----------|--------|------|
| `Integer` / `Float` / `Monetary` | 默认 `sum` | 可通过 `group_operator` 修改 |
| `id` | `count` | 统计记录数 |
| 其他 numeric 字段 | `sum` | 需确保数值有意义 |

> 🔧 **自定义聚合函数**：  
> 在模型中定义字段时设置 `group_operator`：
> ```python
> avg_price = fields.Float(group_operator='avg')
> max_qty = fields.Integer(group_operator='max')
> ```

---

### 五、用户交互功能（自动提供）

| 功能 | 说明 |
|------|------|
| **拖放字段** | 用户可从左侧字段面板拖拽字段到“行”、“列”、“度量”区域 |
| **多度量支持** | 可同时显示多个指标（如数量 + 金额） |
| **层级展开/折叠** | 点击分组标题（如年份）可展开查看子项（如月份） |
| **排序** | 点击列标题可按该列升序/降序排序 |
| **导出** | 支持导出为 CSV（社区版）或 Excel（企业版） |
| **搜索联动** | 受当前 SearchView 的 domain 和 group_by 影响 |

---

### 六、限制与注意事项

1. **无自定义模板**：不能像 kanban 那样使用 QWeb 自定义 HTML。
2. **无按钮/操作**：纯展示视图，不支持 `button` 或 `type="object"`。
3. **性能敏感**：
   - 避免在 `row`/`col` 使用高基数字段（如 thousands of products）。
   - 大数据集建议配合 `domain` 过滤（通过 SearchView）。
4. **空值处理**：`NULL` 值会被归入 `(Undefined)` 分组。
5. **货币格式**：`Monetary` 字段需正确设置 `currency_field`，否则显示为普通数字。

---

## ✅ 最佳实践建议

- **预设常用布局**：在 XML 中定义合理的默认 `row`/`col`/`measure`。
- **优先使用 Many2one/Selection**：作为维度更高效、语义清晰。
- **利用 Date 智能分组**：Odoo 对日期字段自动提供时间层级下钻。
- **命名清晰**：在模型中为聚合字段设置明确的 `string`（如 “Total Revenue”）。
- **测试聚合逻辑**：确保 `group_operator` 符合业务需求（如平均单价用 `avg`）。

---

通过以上配置，你可以在 Odoo 17 中构建灵活、强大的多维分析透视表。如需更复杂指标（如同比、环比、占比），建议在**后端模型中预计算**，再通过 `pivot` 展示结果。