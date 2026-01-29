在 **Odoo 17** 中，`<calendar>` 视图用于以日历形式展示具有时间属性的记录（如会议、任务、日程等）。它支持按天、周、月视图切换，并可拖拽调整时间。

与其它视图不同，`<calendar>` 的核心是 **时间字段**（起始时间、结束时间）和 **显示字段**（标题、颜色等），其结构简洁但属性高度专业化。

---

## ✅ 综合示例：包含所有可用属性的 Calendar 表单（Odoo 17）

```xml
<!-- 文件：views/comprehensive_calendar_view.xml -->
<odoo>
    <record id="view_comprehensive_calendar" model="ir.ui.view">
        <field name="name">comprehensive.calendar</field>
        <field name="model">your.model</field>
        <field name="arch" type="xml">
            <!-- 
                calendar 根元素属性说明：
                - date_start: 必填，事件开始时间字段（datetime 或 date）
                - date_stop: 可选，事件结束时间字段（若无，则用 duration 计算）
                - date_delay: 可选，事件持续时间（小时或天，配合 date_start 使用）
                - all_day: 布尔字段，表示是否为全天事件（影响显示样式）
                - color: 按此字段分组着色（通常为 many2one 或 selection）
                - mode: 默认视图模式（"day" / "week" / "month"）
                - class: 添加 CSS 类
                - js_class: 自定义 JS 控制器（高级用法）
                - create: 是否显示“创建”按钮（UI 层面）
                - delete: 是否允许删除（UI 层面）
                - event_open_popup: 是否点击事件弹出快速表单（而非跳转）
            -->
            <calendar
                date_start="start_datetime"
                date_stop="end_datetime"
                date_delay="duration_hours"
                all_day="is_all_day"
                color="user_id"
                mode="week"
                class="o_comprehensive_calendar"
                js_class="custom_calendar_controller"
                create="true"
                delete="true"
                event_open_popup="true"
            >
                <!-- 
                    字段声明规则：
                    - 所有在卡片中显示的字段必须在此声明
                    - 即使 invisible="1"，只要用于逻辑（如 color）也需声明
                -->

                <!-- 必须的时间字段 -->
                <field name="start_datetime"/>
                <field name="end_datetime"/>
                <field name="duration_hours" invisible="1"/>

                <!-- 全天事件标识 -->
                <field name="is_all_day"/>

                <!-- 颜色分组字段（many2one/selection） -->
                <field name="user_id"/> <!-- 按负责人着色 -->

                <!-- 显示在日历卡片上的字段 -->
                <field name="name"/>          <!-- 事件标题 -->
                <field name="description"/>   <!-- 描述（hover 显示） -->
                <field name="partner_id"/>    <!-- 客户 -->
                <field name="priority"/>      <!-- 优先级（可用于 decoration） -->

                <!-- 用于动态样式的字段（需声明） -->
                <field name="state" invisible="1"/>
                <field name="active" invisible="1"/>

                <!-- 
                    动态行样式（整卡样式）
                    decoration-*: 基于字段值添加样式类
                    注意：calendar 中通过 <attribute> 定义（与 kanban/list 一致）
                -->
                <attribute name="decoration-danger">state == 'urgent'</attribute>
                <attribute name="decoration-warning">priority == 'high'</attribute>
                <attribute name="decoration-muted">not active</attribute>

            </calendar>
        </field>
    </record>
</odoo>
```

> 💡 **关键说明**：
> - **`date_start` 是必填属性**，类型必须为 `date` 或 `datetime`。
> - 若提供 `date_stop`，则忽略 `date_delay`；否则用 `date_start + date_delay` 计算结束时间。
> - `color` 字段决定卡片背景色（Odoo 自动生成配色方案）。
> - `event_open_popup="true"` 时，点击事件会弹出快速编辑表单（类似 kanban 的快速查看）。
> - 所有用于 `decoration-*` 或 `color` 的字段 **必须显式声明**（即使 `invisible="1"`）。

---

## 📚 Calendar 视图所有可操作属性总结

### 一、`<calendar>` 根元素属性

| 属性 | 必填 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `date_start` | ✅ 是 | field name | — | 事件开始时间（`date` 或 `datetime` 字段） |
| `date_stop` | ❌ 否 | field name | — | 事件结束时间（若未设，则用 `date_delay` 计算） |
| `date_delay` | ❌ 否 | field name | — | 持续时间（数值字段，单位由 `field` 的 `widget` 决定） |
| `all_day` | ❌ 否 | field name | — | 布尔字段，标记是否为全天事件 |
| `color` | ❌ 否 | field name | — | 按此字段分组着色（many2one/selection/integer） |
| `mode` | ❌ 否 | `"day"` / `"week"` / `"month"` | `"month"` | 默认日历视图模式 |
| `class` | ❌ 否 | string | — | 添加 CSS 类到日历容器 |
| `js_class` | ❌ 否 | string | — | 指定自定义 JS 控制器（继承 `CalendarController`） |
| `create` | ❌ 否 | boolean | `true` | 是否显示“创建”按钮（UI） |
| `delete` | ❌ 否 | boolean | `true` | 是否允许删除事件（UI） |
| `event_open_popup` | ❌ 否 | boolean | `false` | 点击事件是否弹出快速表单（而非跳转到完整 form） |

> ⚠️ 注意：
> - `date_stop` 和 `date_delay` **二选一**，优先使用 `date_stop`。
> - `all_day` 字段存在时，若值为 `True`，事件将跨越整天（无视具体时间）。

---

### 二、`<field>` 声明（在 `<calendar>` 内）

| 用途 | 说明 |
|------|------|
| **数据声明** | 所有在日历卡片中显示或用于逻辑（如 `color`, `decoration`）的字段必须在此声明 |
| **支持属性** | 仅 `name` 和 `invisible` 有效（`invisible="1"` 用于隐藏但保留数据） |
| **不支持属性** | `widget`, `options`, `readonly`, `required`, `attrs`, `string` 等均无效 |

> 🔍 **字段作用**：
> - `name`：作为事件标题（默认显示）。
> - 其他字段：可在 hover 提示或弹窗中显示（取决于 Odoo 默认模板）。

---

### 三、动态样式：`<attribute name="decoration-*">`

| 属性 | 效果 | 说明 |
|------|------|------|
| `decoration-danger` | 红色边框/文字 | 整个事件卡片添加警示样式 |
| `decoration-warning` | 橙色 | |
| `decoration-success` | 绿色 | |
| `decoration-info` | 蓝色 | |
| `decoration-muted` | 灰色 | 用于非活跃记录 |
| `decoration-bf` | **粗体标题** | |
| `decoration-it` | *斜体* | |

> ✅ 表达式语法：`state == 'urgent'`、`not active`、`priority > 5` 等。  
> 🔧 实现方式：通过 `<attribute>` 标签定义（非直接写在 `<calendar>` 上）。

---

### 四、交互与行为

| 功能 | 说明 |
|------|------|
| **拖拽调整时间** | 拖动事件可修改 `date_start`/`date_stop`（需字段可写） |
| **调整持续时间** | 拖动事件右边缘可修改 `date_delay`（若使用 `date_delay`） |
| **创建事件** | 点击空白时间段可创建新事件（触发快速创建） |
| **点击事件** | 默认跳转到完整表单；若 `event_open_popup="true"` 则弹出快速表单 |
| **颜色分组** | 自动为 `color` 字段的不同值分配唯一颜色 |

---

### 五、支持的字段类型

| 字段类型 | 用途 |
|----------|------|
| `Datetime` | `date_start` / `date_stop`（推荐） |
| `Date` | 仅日期（适合全天事件） |
| `Float` / `Integer` | `date_delay`（持续时间，单位：小时） |
| `Boolean` | `all_day` |
| `Many2one` / `Selection` / `Integer` | `color`（分组着色） |
| `Char` / `Text` | 显示在卡片或 hover 中 |

> 💡 **提示**：若 `date_delay` 字段使用 `widget="float_time"`，则单位为“小时”（如 `2.5` = 2小时30分钟）。

---

### 六、限制与注意事项

1. **性能**：大量事件（>1000）可能导致日历渲染缓慢。
2. **时区**：自动根据用户时区转换显示（后端存储为 UTC）。
3. **权限**：拖拽更新依赖 `date_start`/`date_stop` 字段的写权限。
4. **全天事件**：若 `all_day=True`，则忽略具体时间，事件占据整天。
5. **无自定义模板**：不能像 kanban 那样自定义卡片 HTML（Odoo 使用固定模板）。

---

## ✅ 最佳实践建议

- **必填 `date_start`**：确保模型有合理的开始时间字段。
- **合理选择 `color` 字段**：如负责人（`user_id`）、项目（`project_id`）等。
- **使用 `event_open_popup="true"`**：提升用户体验（避免页面跳转）。
- **声明所有逻辑字段**：包括 `invisible="1"` 的状态字段，用于 `decoration`。
- **测试拖拽行为**：确保时间字段可写且无 domain 限制。

---

## 🗓️ 示例场景（会议日程）

```xml
<calendar
    date_start="start"
    date_stop="stop"
    all_day="allday"
    color="user_id"
    mode="week"
    event_open_popup="true"
>
    <field name="start"/>
    <field name="stop"/>
    <field name="allday"/>
    <field name="user_id"/>
    <field name="name"/>
    <field name="partner_id"/>
    <field name="state" invisible="1"/>
    <attribute name="decoration-danger">state == 'cancel'</attribute>
</calendar>
```

---

通过以上配置，你可以在 Odoo 17 中构建功能完整、交互流畅的日历视图。如需更高级定制（如自定义事件渲染），可通过 `js_class` 注册自定义 OWL 控制器扩展 FullCalendar 库的行为。