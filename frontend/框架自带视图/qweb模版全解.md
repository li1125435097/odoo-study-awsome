在 **Odoo 17** 中，`QWeb` 并不是一个传统意义上的“视图类型”（如 `form`、`list`），而是 Odoo 的 **原生模板引擎**，基于 XML 的声明式模板系统，用于渲染 HTML。

虽然你可以创建一个 `ir.ui.view` 记录并设置 `type="qweb"`，但 **QWeb 模板本身并不直接作为主视图显示在动作（Action）中**（不像 `form` 或 `kanban`）。它主要用于：

- 自定义网页（Website Pages）
- 邮件模板（Email Templates）
- 报表模板（PDF Reports）
- 自定义组件的 OWL 模板（通过 `t-name`）
- 替换或继承现有模板

不过，Odoo 确实允许你注册一个 **独立的 QWeb 模板视图**，并通过控制器（Controller）或特定方式渲染它。

---

## ✅ 完整示例：一个典型的 QWeb 模板 XML（含所有可用属性和指令）

```xml
<!-- 文件：views/custom_qweb_template.xml -->
<odoo>
    <!-- 
        QWeb 模板通常以 <template> 标签定义
        - id: 唯一标识（用于继承或调用）
        - name: 可读名称（可选）
        - t-name: 模板名称（关键！用于 JS 或渲染引擎调用）
        - inherit_id: 用于继承已有模板
        - priority: 继承优先级（数字越小优先级越高，默认为 16）
        - active: 是否启用（默认 True）
    -->
    <template id="custom_dashboard_template" 
              name="Custom Dashboard"
              t-name="CustomDashboard"
              inherit_id="web.webclient_bootstrap"
              priority="10"
              active="True">

        <!-- 
            QWeb 指令（Directives）说明：
            - t-if / t-elif / t-else: 条件渲染
            - t-foreach / t-as: 循环
            - t-esc: 转义输出（安全）
            - t-raw: 不转义输出（危险，慎用）
            - t-att / t-attf: 动态属性
            - t-call: 调用其他模板
            - t-set: 定义变量
            - t-debug: 调试断点（仅开发模式）
        -->

        <!-- 继承：替换 webclient 的主容器 -->
        <xpath expr="//div[@id='web_client']" position="replace">
            <div id="custom_dashboard" class="o_custom_dashboard">
                <h1 t-esc="title"/> <!-- 安全输出标题 -->

                <!-- 条件渲染 -->
                <div t-if="user.has_group('base.group_system')">
                    <p>Admin Panel</p>
                </div>

                <!-- 循环渲染列表 -->
                <ul>
                    <li t-foreach="records" t-as="record">
                        <span t-att-title="record.description"> <!-- 动态 title 属性 -->
                            <t t-esc="record.name"/>
                        </span>
                        <!-- 使用 t-attf 支持字符串插值 -->
                        <span t-attf-class="status-#{record.state}"/>
                    </li>
                </ul>

                <!-- 调用子模板 -->
                <t t-call="CustomDashboard.Footer"/>

                <!-- 定义变量 -->
                <t t-set="current_year" t-value="datetime.datetime.now().year"/>
                <p>© <t t-esc="current_year"/> My Company</p>

                <!-- 调试（仅当 odoo 以 --dev=all 启动时生效） -->
                <t t-debug=""/>
            </div>
        </xpath>
    </template>

    <!-- 子模板：Footer -->
    <template t-name="CustomDashboard.Footer">
        <footer class="o_footer">
            <p>Built with Odoo</p>
        </footer>
    </template>

    <!-- 
        注意：QWeb 模板也可以不继承，直接定义完整 HTML（用于 Website）
        示例：自定义网页
    -->
    <template id="website_custom_page" name="Custom Web Page">
        <t t-call="website.layout"> <!-- 继承网站基础布局 -->
            <div class="oe_structure">
                <section class="container">
                    <h2>Welcome to Our Service</h2>
                    <p t-raw="dynamic_content"/> <!-- 渲染富文本（来自后台） -->
                </section>
            </div>
        </t>
    </template>
</odoo>
```

> 💡 **关键说明**：
> - **`t-name` 是核心属性**：JS 或 Python 渲染时通过此名称调用模板。
> - **`inherit_id` 用于模板继承**：类似视图继承，使用 `<xpath>` 修改原模板。
> - **QWeb 指令以 `t-` 开头**：这是 Odoo 模板引擎的语法。
> - **不能直接通过 Action 打开 QWeb 模板**：需通过控制器返回 `request.render("t-name", values)`。
> - **`<template>` 不是 `<qweb>`**：Odoo 社区文档中常称其为 “QWeb template”，但 XML 标签是 `<template>`。

---

## 📚 QWeb 模板所有可操作属性与指令总结

### 一、`<template>` 元素属性（用于 `ir.ui.view` 注册）

| 属性 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `id` | ✅ 是 | — | XML ID（用于继承或引用） |
| `t-name` | ⚠️ 视用途而定 | — | **模板名称**，JS/Python 渲染时的关键标识 |
| `inherit_id` | ❌ 否 | — | 被继承的模板 ID（如 `"web.login"`） |
| `priority` | ❌ 否 | `16` | 继承优先级（数值越小越先应用） |
| `active` | ❌ 否 | `True` | 是否启用该模板 |
| `name` | ❌ 否 | — | 可读名称（仅用于管理界面） |

> 🔍 **注意**：
> - 如果不设 `t-name`，模板无法被 JS 或 Python 直接调用。
> - `inherit_id` 和 `t-name` 通常**不同时使用**：  
>   - 继承模板 → 设 `inherit_id`，**不设 `t-name`**  
>   - 独立模板 → 设 `t-name`，**不设 `inherit_id`**

---

### 二、QWeb 指令（T-directives）

| 指令 | 说明 | 示例 |
|------|------|------|
| `t-if` | 条件渲染 | `<div t-if="user.admin">Admin</div>` |
| `t-elif` / `t-else` | 条件分支 | `<t t-elif="...">` / `<t t-else="">` |
| `t-foreach` + `t-as` | 循环 | `<li t-foreach="items" t-as="item">` |
| `t-esc` | 转义输出（安全） | `<span t-esc="name"/>` |
| `t-raw` | 不转义输出（危险） | `<div t-raw="html_content"/>` |
| `t-att` | 动态属性（字典） | `<div t-att="{'class': my_class, 'id': my_id}">` |
| `t-attf` | 动态属性（字符串插值） | `<div t-attf-class="btn-#{state}">` |
| `t-att-<attribute>` | 单属性动态绑定 | `<div t-att-class="my_class">` |
| `t-call` | 调用其他模板 | `<t t-call="MySubTemplate"/>` |
| `t-set` + `t-value` | 定义变量 | `<t t-set="x" t-value="10"/>` |
| `t-debug` | 调试断点 | `<t t-debug=""/>`（需 `--dev=all`） |
| `t-out` | （Odoo 15+）类似 `t-esc`，支持格式化 | `<span t-out="amount" t-options='{"widget": "monetary"}'/>` |

> ⚠️ **安全警告**：
> - **永远不要对用户输入使用 `t-raw`**，除非你完全信任内容（如后台富文本）。
> - 优先使用 `t-esc` 或 `t-out`。

---

### 三、循环变量（在 `t-foreach` 中自动可用）

当使用 `<t t-foreach="items" t-as="item">` 时，以下变量自动可用：

| 变量 | 说明 |
|------|------|
| `item` | 当前元素 |
| `item_index` | 索引（从 0 开始） |
| `item_size` | 总数量 |
| `item_first` | 是否第一个（布尔） |
| `item_last` | 是否最后一个（布尔） |
| `item_parity` | `"even"` 或 `"odd"` |

示例：
```xml
<li t-foreach="users" t-as="user" t-att-class="user_parity">
    <t t-if="user_first">First: </t>
    <t t-esc="user.name"/>
</li>
```

---

### 四、模板继承机制

- 使用 `<xpath expr="..." position="...">` 修改原模板。
- `position` 可选值：`replace`, `inside`, `before`, `after`, `attributes`。
- 可多重继承（按 `priority` 排序）。

示例（添加 CSS 类）：
```xml
<xpath expr="//body" position="attributes">
    <attribute name="class">custom-body</attribute>
</xpath>
```

---

### 五、使用场景总结

| 场景 | 如何使用 |
|------|----------|
| **自定义网页** | 定义 `<template>` + `t-call="website.layout"`，通过 URL 访问 |
| **邮件模板** | 在 `mail.template` 中使用 QWeb 语法 |
| **PDF 报表** | 定义 `report.paperformat` + QWeb 模板 |
| **替换 Odoo 界面** | 继承 `web.login`、`web.webclient_bootstrap` 等 |
| **OWL 组件模板** | 在 JS 组件中通过 `static template = xml` 或 `t-name` 引用 |

---

### 六、限制与注意事项

1. **不能作为主视图**：无法通过 `ir.actions.act_window` 直接打开 QWeb 模板。
2. **需控制器配合**：独立页面需 Python Controller 返回 `request.render()`。
3. **无内置数据获取**：所有数据必须由后端传入（`values` 字典）。
4. **性能敏感**：避免在循环中执行复杂逻辑（QWeb 是服务端渲染）。
5. **调试困难**：错误信息不直观，建议开启 `--dev=qweb` 查看编译后代码。

---

## ✅ 最佳实践建议

- **模块化模板**：将大模板拆分为多个 `t-call` 子模板。
- **避免逻辑复杂化**：业务逻辑放在 Python，模板只负责展示。
- **使用 `t-out` 替代 `t-esc`**（Odoo 15+）：支持字段 widget 渲染（如货币、日期）。
- **继承而非重写**：优先使用 `inherit_id` 修改官方模板。
- **命名规范**：`t-name="ModuleName.TemplateName"` 避免冲突。

---

## 🌐 示例：通过控制器渲染 QWeb 模板

```python
# controllers/main.py
from odoo import http
from odoo.http import request

class CustomPage(http.Controller):
    @http.route('/my-dashboard', auth='user')
    def dashboard(self):
        records = request.env['your.model'].search([])
        return request.render('your_module.CustomDashboard', {
            'title': 'My Dashboard',
            'records': records,
            'user': request.env.user,
        })
```

然后访问 `/my-dashboard` 即可看到渲染结果。

---

通过以上配置，你可以在 Odoo 17 中灵活使用 QWeb 模板构建自定义界面。记住：**QWeb 是 Odoo 的“HTML 画布”**，结合后端数据和前端指令，能实现高度定制化的用户体验。