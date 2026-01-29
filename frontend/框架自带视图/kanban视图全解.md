在 **Odoo 17** 中，看板视图（Kanban View）用于以卡片形式可视化记录，常用于项目管理、任务跟踪、销售机会等场景。它高度可定制，支持拖拽分组、自定义模板、颜色标记、快速操作等。

以下将：

1. **提供一个包含几乎所有可用属性的完整 `<kanban>` 视图 XML 示例（带详细注释）**  
2. **系统总结所有可操作属性及其作用**

---

## ✅ 综合示例：包含所有常用属性的 Kanban 表单（Odoo 17）

```xml
<!-- 文件：views/comprehensive_kanban_view.xml -->
<odoo>
    <record id="view_comprehensive_kanban" model="ir.ui.view">
        <field name="name">comprehensive.kanban</field>
        <field name="model">your.model</field>
        <field name="arch" type="xml">
            <!-- 
                kanban 根元素属性说明：
                - default_group_by: 默认按某字段分组（必须是 many2one / selection / boolean）
                - default_order: 卡片默认排序
                - class: 添加 CSS 类
                - js_class: 指定自定义 JS 控制器（高级用法）
                - create: 是否显示“创建”按钮（UI 层面）
                - quick_create: 是否启用快速创建（+ 按钮）
                - quick_create_view: 指定快速创建使用的 form 视图 ID
                - on_create: 自定义创建行为（如打开特定 Action）
                - group_create: 是否允许创建新分组（如新阶段）
                - group_edit: 是否允许编辑分组标题
                - group_delete: 是否允许删除空分组
                - limit: 每组加载卡片数（默认 40）
                - default_col_number: 默认列数（已弃用，由分组决定）
            -->
            <kanban
                default_group_by="stage_id"
                default_order="priority desc, id asc"
                class="o_comprehensive_kanban"
                js_class="custom_kanban_controller"
                create="true"
                quick_create="true"
                quick_create_view="my_module.view_your_model_form_simple"
                on_create="action_open_custom_form"
                group_create="true"
                group_edit="true"
                group_delete="true"
                limit="30"
            >
                <!-- 
                    字段声明：所有在模板中使用的字段必须在此声明（即使 invisible）
                    否则无法访问数据。
                -->
                <field name="id"/>
                <field name="name"/>
                <field name="description"/>
                <field name="priority"/>
                <field name="color"/>
                <field name="stage_id"/>
                <field name="user_id"/>
                <field name="deadline"/>
                <field name="is_overdue" invisible="1"/>
                <field name="tag_ids"/>
                <field name="company_id" invisible="1"/>

                <!-- 
                    templates: 定义卡片和分组的渲染模板
                    - t-name="kanban-box": 每张卡片的模板（必填）
                    - t-name="kanban-card": （Odoo 17 中通常用 kanban-box）
                    - t-name="kanban-header": 分组标题模板（可选）
                -->
                <templates>
                    <!-- 
                        每张卡片的模板
                        - record: 当前记录对象（通过 record.data 访问字段值）
                        - widget: 提供工具方法（如 getColSpan）
                    -->
                    <t t-name="kanban-box">
                        <div 
                            t-attf-class="oe_kanban_card oe_kanban_global_click 
                                #{record.color.raw_value ? 'bg-' + record.color.raw_value : ''}"
                        >
                            <!-- 卡片顶部：颜色条/标签 -->
                            <div class="oe_kanban_card_header">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title">
                                        <t t-esc="record.name.value"/>
                                    </strong>
                                    <span 
                                        t-if="record.user_id.value" 
                                        class="o_kanban_record_subtitle"
                                    >
                                        Assigned to: <t t-esc="record.user_id.value"/>
                                    </span>
                                </div>
                                <!-- 右上角操作菜单（...） -->
                                <div class="o_kanban_record_dropdown">
                                    <a 
                                        role="button" 
                                        class="dropdown-toggle" 
                                        data-bs-toggle="dropdown"
                                    >
                                        <i class="fa fa-ellipsis-v"/>
                                    </a>
                                    <ul class="dropdown-menu" role="menu">
                                        <li>
                                            <a 
                                                role="button" 
                                                type="edit"
                                                class="dropdown-item"
                                            >Edit</a>
                                        </li>
                                        <li>
                                            <a 
                                                role="button" 
                                                name="action_archive"
                                                type="object"
                                                class="dropdown-item text-danger"
                                            >Archive</a>
                                        </li>
                                    </ul>
                                </div>
                            </div>

                            <!-- 卡片主体 -->
                            <div class="oe_kanban_details">
                                <ul class="mb-0">
                                    <li t-if="record.description.value">
                                        <t t-esc="record.description.value"/>
                                    </li>
                                    <li t-if="record.deadline.value">
                                        <i class="fa fa-clock-o me-1"/>
                                        Deadline: <t t-esc="record.deadline.value"/>
                                        <t t-if="record.is_overdue.value">
                                            <span class="text-danger">(Overdue)</span>
                                        </t>
                                    </li>
                                    <!-- many2many_tags 渲染 -->
                                    <li t-if="record.tag_ids.value">
                                        <t t-foreach="record.tag_ids.value" t-as="tag">
                                            <span 
                                                t-attf-class="badge bg-{{ tag.color or 'secondary' }} me-1"
                                            >
                                                <t t-esc="tag.display_name"/>
                                            </span>
                                        </t>
                                    </li>
                                </ul>
                            </div>

                            <!-- 底部快速按钮（可选） -->
                            <div class="oe_kanban_footer">
                                <button 
                                    name="action_mark_done" 
                                    type="object"
                                    class="btn btn-sm btn-primary"
                                >
                                    Mark Done
                                </button>
                            </div>
                        </div>
                    </t>

                    <!-- 
                        自定义分组标题模板（可选）
                        - group: 分组对象（group.value = 分组字段值）
                    -->
                    <t t-name="kanban-header">
                        <div class="o_kanban_header">
                            <span t-esc="group.value"/>
                            <span class="o_kanban_header_count badge bg-secondary ms-2">
                                <t t-esc="group.count"/>
                            </span>
                        </div>
                    </t>
                </templates>

                <!-- 
                    动态行样式（整卡样式）
                    decoration-*: 基于字段值添加样式类
                -->
                <field name="priority" invisible="1"/>
                <field name="state" invisible="1"/>
                <field name="active" invisible="1"/>

                <!-- decoration-danger 等会为整个 .oe_kanban_card 添加类 -->
                <attribute name="decoration-danger">priority == 'high'</attribute>
                <attribute name="decoration-warning">state == 'pending'</attribute>
                <attribute name="decoration-muted">not active</attribute>
                <attribute name="decoration-bf">is_important == True</attribute>

            </kanban>
        </field>
    </record>
</odoo>
```

> 💡 **关键说明**：
> - 所有在 QWeb 模板中使用的字段 **必须在 `<kanban>` 内通过 `<field>` 声明**，否则 `record.field` 为 `undefined`。
> - `t-attf-class` 支持字符串插值（如 `bg-{{ color }}`）。
> - `decoration-*` 属性需通过 `<attribute>` 标签定义（与 list/form 不同）。
> - 快速创建（`quick_create`）默认使用简化表单，可通过 `quick_create_view` 指定。

---

## 📚 Kanban 视图所有可操作属性总结

### 一、`<kanban>` 根元素属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `default_group_by` | field name | — | 默认按该字段分组（必须支持分组：many2one/selection/boolean） |
| `default_order` | string | 模型 `_order` | 卡片默认排序（如 `"priority desc, id"`） |
| `class` | string | — | 添加 CSS 类到整个 kanban 容器 |
| `js_class` | string | — | 指定自定义 JS 控制器（继承 `KanbanController`） |
| `create` | boolean | `true` | 是否显示“创建”按钮（UI） |
| `quick_create` | boolean | `true` | 是否启用快速创建（+ 按钮） |
| `quick_create_view` | view id | — | 指定快速创建使用的 form 视图（如 `my_module.my_form`） |
| `on_create` | action/method | — | 自定义创建行为（如打开特定 Action） |
| `group_create` | boolean | `true` | 是否允许创建新分组（如输入新阶段名） |
| `group_edit` | boolean | `true` | 是否允许编辑分组标题（点击分组名修改） |
| `group_delete` | boolean | `true` | 是否允许删除空分组 |
| `limit` | integer | `40` | 每组默认加载卡片数 |

> ⚠️ 注意：`editable`、`delete` 等属性 **不适用于 kanban**。

---

### 二、`<field>` 声明（在 `<kanban>` 内）

| 用途 | 说明 |
|------|------|
| **数据声明** | 所有在模板中使用的字段必须在此声明（即使 `invisible="1"`） |
| **支持属性** | `name`（必填）、`invisible`（隐藏但保留数据） |
| **不支持属性** | `widget`, `options`, `readonly`, `required`, `attrs` 等（这些在 kanban 模板中通过 JS/OWL 处理） |

---

### 三、`<templates>` 内模板

#### 1. `t-name="kanban-box"`（必填）
- 定义每张卡片的 HTML 结构。
- 使用 `record.field.value` 获取显示值，`record.field.raw_value` 获取原始值。
- 支持 QWeb 指令：`t-if`, `t-foreach`, `t-esc`, `t-attf-class` 等。

#### 2. `t-name="kanban-header"`（可选）
- 自定义分组标题栏。
- 使用 `group.value`（分组值）、`group.count`（记录数）。

---

### 四、动态样式：`<attribute name="decoration-*">`

| 属性 | 效果 | 说明 |
|------|------|------|
| `decoration-danger` | 红色边框/文字 | 整卡添加 `text-danger` 或自定义逻辑 |
| `decoration-warning` | 橙色 | |
| `decoration-success` | 绿色 | |
| `decoration-info` | 蓝色 | |
| `decoration-muted` | 灰色 | |
| `decoration-bf` | **粗体** | |
| `decoration-it` | *斜体* | |

> ✅ 表达式语法：`priority == 'high'`、`not active`、`deadline < current_date` 等。  
> 🔧 实现方式：通过 `<attribute>` 标签定义（非直接写在 `<kanban>` 上）。

---

### 五、交互与操作

| 元素 | 说明 |
|------|------|
| **全局点击** | 添加 `oe_kanban_global_click` 类，点击卡片任意位置打开表单 |
| **编辑菜单** | 通过 `.o_kanban_record_dropdown` 实现标准“...”菜单 |
| **按钮** | 在模板中使用 `<button name="method" type="object"/>` 触发 Python 方法 |
| **拖拽** | 自动支持跨组拖拽（需 `default_group_by` 字段可写） |

---

### 六、其他注意事项

1. **性能优化**：
   - 避免在卡片中加载大字段（如 `html`、`binary`）。
   - 使用 `limit` 控制初始加载量。

2. **权限控制**：
   - 拖拽更新依赖字段写权限。
   - 按钮操作受方法权限控制。

3. **响应式**：
   - 卡片宽度自动适配（默认 250px~300px）。
   - 移动端支持滑动查看分组。

---

## ✅ 最佳实践建议

- **最小化字段声明**：只声明模板实际使用的字段，提升性能。
- **使用 `oe_kanban_global_click`**：提升用户体验（点击即打开）。
- **合理使用颜色**：通过 `record.color.raw_value` 动态设置背景（需字段为 `integer` 或 `char` 颜色名）。
- **避免复杂逻辑**：复杂计算应在 Python 中完成，前端只负责展示。
- **测试拖拽行为**：确保分组字段可写且无 domain 限制。

---

通过以上配置，你可以在 Odoo 17 中构建功能丰富、交互流畅的看板视图。如需实现更高级功能（如自定义拖拽规则、动态分组），可通过 `js_class` 注册自定义 OWL 控制器扩展。