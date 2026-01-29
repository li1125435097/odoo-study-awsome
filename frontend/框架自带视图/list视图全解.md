在 **Odoo 17** 中，列表视图（List View）用于以表格形式展示记录集合。自 Odoo 16 起，官方推荐使用 `<list>` 标签（替代旧版 `<tree>`，尽管后者仍兼容）。列表视图支持丰富的属性来控制排序、分组、编辑行为、样式等。

以下将：

1. **提供一个包含几乎所有可用属性的完整 `<list>` 视图 XML 示例**（带详细注释）  
2. **系统总结所有可操作属性及其作用**

---

## ✅ 综合示例：包含所有常用属性的 List 表单（Odoo 17）

```xml
<!-- 文件：views/comprehensive_list_view.xml -->
<odoo>
    <record id="view_comprehensive_list" model="ir.ui.view">
        <field name="name">comprehensive.list</field>
        <field name="model">your.model</field>
        <field name="arch" type="xml">
            <!-- 
                list 根元素属性说明：
                - editable: 是否可内联编辑（"top" / "bottom" / 不设则只读）
                - default_order: 默认排序字段（支持多字段和方向）
                - default_group_by: 默认按某字段分组（需该字段有 group_operator 或为 many2one/selection）
                - create: 是否显示“创建”按钮（UI 层面）
                - delete: 是否允许删除（UI 层面）
                - js_class: 指定自定义 JS 控制器（高级用法）
                - class: 添加 CSS 类
                - limit: 每页默认加载记录数（默认 80）
                - decoration-*: 基于字段值设置行样式（如颜色、粗体等）
            -->
            <list 
                editable="top"
                default_order="name asc, priority desc"
                default_group_by="category_id"
                create="true"
                delete="true"
                js_class="custom_list_controller"
                class="o_comprehensive_list"
                limit="50"
                decoration-danger="priority == 'high'"
                decoration-warning="state == 'pending'"
                decoration-muted="active == False"
                decoration-bf="is_important == True"        <!-- bold -->
                decoration-it="is_archived == True"         <!-- italic -->
            >
                <!-- 
                    字段列定义：
                    - name: 必填，模型字段名
                    - string: 覆盖列标题
                    - invisible: 静态隐藏（仍存在于 DOM）
                    - readonly: 是否可编辑（仅在 editable 模式下有效）
                    - required: 是否必填（仅在 editable 模式下有效）
                    - widget: 指定前端组件（如 many2many_tags, monetary, url 等）
                    - options: 传递配置给 widget
                    - domain/context: 用于 many2one/many2many 过滤或上下文
                    - sum: 显示列底部合计（需字段为 numeric 且有 group_operator）
                    - avg/min/max: 类似 sum，但较少用
                    - optional: 是否默认隐藏（用户可手动显示）
                    - widget_onclick: 自定义点击行为（极少用）
                -->

                <!-- 主键字段（通常不可见，但建议保留） -->
                <field name="id" invisible="1"/>

                <!-- 常规字段 -->
                <field 
                    name="name" 
                    string="Document Name" 
                    required="1"
                    readonly="0"
                />

                <!-- 带 widget 和 options 的字段 -->
                <field 
                    name="tag_ids" 
                    widget="many2many_tags" 
                    options="{'color_field': 'color', 'no_create': True}"
                />

                <!-- 货币字段 -->
                <field name="currency_id" invisible="1"/>
                <field 
                    name="amount_total" 
                    widget="monetary" 
                    options="{'currency_field': 'currency_id'}"
                    sum="Total Amount"  <!-- 底部显示合计 -->
                />

                <!-- URL 字段 -->
                <field 
                    name="website_url" 
                    widget="url"
                />

                <!-- 只读计算字段（force_save 无效于 list） -->
                <field 
                    name="computed_status" 
                    readonly="1"
                />

                <!-- 条件隐藏字段（用户可选显示） -->
                <field 
                    name="internal_note" 
                    optional="1"  <!-- 默认隐藏，用户可通过列选择器显示 -->
                />

                <!-- 用于分组和排序的字段（即使 invisible 也可用于 default_group_by） -->
                <field name="category_id" invisible="1"/>
                <field name="priority" invisible="1"/>
                <field name="state" invisible="1"/>
                <field name="active" invisible="1"/>
                <field name="is_important" invisible="1"/>
                <field name="is_archived" invisible="1"/>

                <!-- 按钮列（在 list 中较少见，但支持） -->
                <!-- 注意：button 在 editable 模式下可能位置异常，通常放 form -->
                <button 
                    name="action_quick_view" 
                    type="object" 
                    string="View" 
                    icon="fa-eye"
                    class="btn-link"
                />

            </list>
        </field>
    </record>
</odoo>
```

> 💡 **说明**：
> - 此 XML 假设模型 `your.model` 已定义相应字段（如 `name`, `tag_ids`, `amount_total` 等）。
> - `editable="top"` 表示新行插入在顶部；`"bottom"` 插入在底部。
> - `decoration-*` 属性会根据字段值动态为整行添加 Bootstrap 样式类（如 `text-danger`）。
> - `optional="1"` 字段默认不显示，但用户可通过列表右上角 “•••” → “Customize” → 勾选显示。

---

## 📚 List 视图所有可操作属性总结

### 一、`<list>` 根元素属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `editable` | `"top"` / `"bottom"` | — | 启用内联编辑，新行插入位置 |
| `default_order` | string | 模型 `_order` | 默认排序（如 `"name asc, id desc"`） |
| `default_group_by` | field name | — | 默认按该字段分组（需支持分组） |
| `create` | boolean | `true` | 是否显示“创建”按钮（UI） |
| `delete` | boolean | `true` | 是否允许删除（UI） |
| `js_class` | string | — | 指定自定义 JS 控制器类（高级） |
| `class` | string | — | 添加 CSS 类 |
| `limit` | integer | `80` | 每页加载记录数 |
| `decoration-<style>` | expression | — | 动态行样式（见下表） |

#### `decoration-*` 支持的样式：

| 属性 | 效果 | 对应 CSS 类 |
|------|------|-------------|
| `decoration-danger` | 红色文字 | `text-danger` |
| `decoration-warning` | 橙色文字 | `text-warning` |
| `decoration-success` | 绿色文字 | `text-success` |
| `decoration-info` | 蓝色文字 | `text-info` |
| `decoration-muted` | 灰色文字 | `text-muted` |
| `decoration-bf` | **粗体** | `fw-bold` |
| `decoration-it` | *斜体* | `fst-italic` |

> ✅ 表达式语法：`field_name == 'value'`、`priority > 5`、`not active` 等。

---

### 二、`<field>` 元素属性（在 `<list>` 中）

| 属性 | 说明 |
|------|------|
| `name` | **必填**，模型字段名 |
| `string` | 覆盖列标题 |
| `invisible` | 静态隐藏列（仍可用于排序/分组） |
| `readonly` | 在 `editable` 模式下是否可编辑 |
| `required` | 在 `editable` 模式下是否必填 |
| `widget` | 指定前端组件（如 `many2many_tags`, `monetary`, `url`, `phone`） |
| `options` | JSON 配置（如 `{"no_open": true}`） |
| `domain` | 过滤关联记录（对 many2one/many2many 有效） |
| `context` | 传递上下文（如 `{'default_user_id': uid}`） |
| `sum` | 在列底部显示合计（需 numeric 字段） |
| `avg` / `min` / `max` | 显示平均值/最小值/最大值（较少用） |
| `optional` | 默认隐藏，用户可手动启用（通过列选择器） |
| `class` | 添加 CSS 类（如 `text-end` 右对齐） |

> ⚠️ 注意：
> - `force_save`、`password`、`nolabel` 等属性 **在 list 中无效**。
> - `attrs` **不支持** 在 list 的 `<field>` 中使用（仅 form 支持）。

---

### 三、`<button>` 元素属性（在 `<list>` 中）

虽然不常见，但 list 支持按钮列：

| 属性 | 说明 |
|------|------|
| `name` | 方法名或 Action ID |
| `type` | `object` / `action` |
| `string` | 按钮文本 |
| `icon` | 图标（如 `fa-print`） |
| `class` | 如 `btn-link`（无背景按钮） |
| `confirm` | 确认对话框文本 |

> ❗ 限制：按钮在 `editable` 模式下可能布局错乱，建议仅用于只读列表。

---

### 四、其他注意事项

1. **分组（Group By）要求**：
   - 字段必须是 `many2one`、`selection`、`date`、`datetime` 或定义了 `group_operator` 的 numeric 字段。
   - 分组后自动显示聚合（如 count、sum）。

2. **性能优化**：
   - 避免在 list 中加载大字段（如 `html`、`binary`）。
   - 使用 `invisible="1"` 隐藏仅用于逻辑的字段。

3. **用户自定义**：
   - 用户可通过界面调整列顺序、宽度、可见性，Odoo 会保存到 `ir.ui.view.custom`。

---

## ✅ 最佳实践建议

- **默认隐藏非关键字段**：使用 `optional="1"` 提升可读性。
- **合理使用 `editable`**：仅当需要快速批量编辑时启用。
- **货币字段必配 `currency_field`**：避免金额显示错误。
- **避免复杂 widget**：如 `image`、`text` 不适合列表，用 `char` 摘要替代。
- **利用 `decoration-*`**：直观传达状态（如高优先级标红）。

---

通过以上结构化配置，你可以在 Odoo 17 中构建功能强大、用户体验良好的列表视图。如需实现更复杂交互（如自定义列渲染、拖拽排序），可通过 `js_class` 注册自定义 OWL 控制器扩展。