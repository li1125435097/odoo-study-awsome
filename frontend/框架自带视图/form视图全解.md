在 **Odoo 17** 中，`<form>` 视图是用户与记录交互的核心界面。虽然并非所有属性都能“同时”有意义地用在一个字段上（例如 `password="1"` 只适用于 `char` 字段），但我们可以构造一个**高度综合的示例 XML 表单**，尽可能涵盖所有**常用且有效的可操作属性**，并附上详细注释。

---

## ✅ 综合示例：包含几乎所有可用属性的 Form 表单（Odoo 17）

```xml
<!-- 文件：views/comprehensive_form_view.xml -->
<odoo>
    <record id="view_comprehensive_form" model="ir.ui.view">
        <field name="name">comprehensive.form</field>
        <field name="model">your.model</field>
        <field name="arch" type="xml">
            <!-- 
                form 根元素属性说明：
                - edit: 控制是否可编辑（UI 层面）
                - create: 是否允许创建（通常由 Action 控制，此处可覆盖）
                - delete: 是否显示删除按钮
                - duplicate: 是否显示复制按钮
                - js_class: 指定自定义 JS 控制器（高级用法）
                - class: 添加 CSS 类
            -->
            <form 
                edit="true" 
                create="true" 
                delete="true" 
                duplicate="true"
                js_class="custom_form_controller"
                class="o_comprehensive_form"
            >
                <!-- header 区域：放置状态栏、主操作按钮 -->
                <header>
                    <!-- 状态栏字段（通常为 selection 字段） -->
                    <field 
                        name="state" 
                        widget="statusbar" 
                        statusbar_visible="draft,confirmed,done"
                        clickable="True"
                    />
                    <!-- 主操作按钮 -->
                    <button 
                        name="action_confirm" 
                        type="object" 
                        string="Confirm" 
                        class="oe_highlight"
                        icon="fa-check"
                        confirm="Are you sure you want to confirm?"
                        context="{'from_ui': True}"
                        attrs="{'invisible': [('state', '!=', 'draft')]}"
                    />
                </header>

                <!-- sheet：表单主体内容容器 -->
                <sheet>

                    <!-- 分隔标题 -->
                    <separator string="Basic Information"/>

                    <!-- group：默认两列布局 -->
                    <group col="4"> <!-- col="4" 表示每行4个字段 -->

                        <!-- 普通字段：展示 string 覆盖、readonly、required、invisible -->
                        <field 
                            name="name" 
                            string="Document Name" 
                            required="1"
                            readonly="0"
                            invisible="0"
                            class="my_name_field"
                        />

                        <!-- 使用 widget 和 options -->
                        <field 
                            name="color" 
                            widget="color_picker" 
                            options="{'palette': ['red', 'green', 'blue']}"
                        />

                        <!-- 密码字段（仅限 char 字段） -->
                        <field 
                            name="api_key" 
                            password="1"
                            nolabel="1" <!-- 不显示标签 -->
                        />

                        <!-- 带 domain 和 context 的 many2one -->
                        <field 
                            name="partner_id" 
                            domain="[('is_company', '=', True)]"
                            context="{'show_address': 1, 'default_customer_rank': 1}"
                            options="{'no_open': True, 'no_create': False}"
                        />

                        <!-- 动态控制：使用 attrs 实现响应式 UI -->
                        <field 
                            name="delivery_required" 
                            type="boolean"
                        />
                        <field 
                            name="delivery_date" 
                            attrs="{
                                'invisible': [('delivery_required', '=', False)],
                                'required': [('delivery_required', '=', True)]
                            }"
                        />

                        <!-- monetary 字段需配合 currency_field -->
                        <field name="currency_id" invisible="1"/>
                        <field 
                            name="amount_total" 
                            widget="monetary" 
                            options="{'currency_field': 'currency_id'}"
                        />

                        <!-- 图片字段 -->
                        <field 
                            name="image" 
                            widget="image" 
                            class="o_image_field"
                            options="{'size': [128, 128]}"
                        />

                        <!-- URL 字段 -->
                        <field 
                            name="website_url" 
                            widget="url"
                        />

                        <!-- 强制保存（即使未修改） -->
                        <field 
                            name="computed_field" 
                            readonly="1"
                            force_save="1" <!-- 用于确保只读字段也能提交 -->
                        />

                    </group>

                    <!-- Notebook (Tabbed interface) -->
                    <notebook>

                        <!-- 第一个 Tab -->
                        <page string="Details" name="details_page">
                            <group>
                                <field name="description" nolabel="1"/>
                            </group>
                        </page>

                        <!-- 带条件的 Tab -->
                        <page 
                            string="Advanced Settings" 
                            name="advanced_page"
                            attrs="{'invisible': [('user_can_edit_advanced', '=', False)]}"
                        >
                            <group>
                                <field name="debug_mode" readonly="1"/>
                            </group>
                        </page>

                    </notebook>

                    <!-- 自定义组件（非字段） -->
                    <div t-component="CustomDashboardWidget"/>

                </sheet>
            </form>
        </field>
    </record>
</odoo>
```

> 💡 **说明**：
> - 此 XML 假设模型 `your.model` 已定义相应字段（如 `name`, `state`, `partner_id`, `image` 等）。
> - `t-component="CustomDashboardWidget"` 需要你在 JS 中注册该 OWL 组件（见前文）。
> - 并非所有属性在所有字段类型上都有效（如 `password` 仅对 `char` 有效）。

---

## 📚 所有可操作属性总结（按元素分类）

### 一、`<form>` 根元素属性

| 属性 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `edit` | boolean | `true` | 是否允许编辑（UI 层面） |
| `create` | boolean | `true` | 是否允许创建（UI） |
| `delete` | boolean | `true` | 是否显示删除按钮 |
| `duplicate` | boolean | `true` | 是否显示复制按钮 |
| `js_class` | string | — | 指定自定义 JS 控制器类（高级） |
| `class` | string | — | 添加 CSS 类 |

---

### 二、`<field>` 元素属性（最核心）

| 属性 | 说明 |
|------|------|
| `name` | **必填**，模型字段名 |
| `string` | 覆盖字段标签文本 |
| `readonly` | 静态只读（布尔值） |
| `required` | 静态必填（布尔值） |
| `invisible` | 静态隐藏（仍存在于 DOM） |
| `class` | 添加 CSS 类 |
| `widget` | 指定前端组件（如 `image`, `url`, `many2many_tags`） |
| `options` | JSON 对象，传递配置给 widget |
| `domain` | 过滤关联记录（支持上下文变量） |
| `context` | 传递上下文给后端（如默认值、行为标志） |
| `attrs` | **动态控制** `readonly`/`required`/`invisible`（推荐） |
| `force_save` | 即使未修改也提交该字段（常用于只读计算字段） |
| `nolabel` | 不显示字段标签（常用于大文本或图片） |
| `password` | 将输入框转为密码类型（仅 `char` 字段） |
| `colspan` | 跨越列数（在 `<group>` 中使用） |
| `placeholder` | 输入提示文本（部分 widget 支持） |

> ⚠️ 注意：`readonly`/`required`/`invisible` **优先级**：`attrs` > 静态属性。

---

### 三、`<button>` 属性

| 属性 | 说明 |
|------|------|
| `name` | 方法名（`type="object"`）或 Action ID |
| `type` | `object` / `action` / `reset` / `url` |
| `string` | 按钮文本 |
| `icon` | Font Awesome 图标（如 `fa-print`） |
| `class` | 如 `oe_highlight`（主按钮样式） |
| `confirm` | 点击前弹出确认对话框 |
| `context` | 传递额外上下文 |
| `attrs` | 动态控制按钮可见性等 |
| `invisible` | 静态隐藏 |

---

### 四、`<group>` 属性

| 属性 | 说明 |
|------|------|
| `col` | 列数（默认 `2`） |
| `class` | CSS 类 |

---

### 五、`<page>`（Notebook Tab）属性

| 属性 | 说明 |
|------|------|
| `string` | 标签名（必填） |
| `name` | 用于 JS 或引用 |
| `invisible` / `attrs` | 控制是否显示该页 |
| `class` | CSS 类 |

---

### 六、`<separator>` 属性

| 属性 | 说明 |
|------|------|
| `string` | 显示为小标题 |
| `colspan` | 跨越列数（默认 `2`） |

---

### 七、其他

- `<header>`：无特殊属性，用于放置状态栏和主按钮。
- `<sheet>`：建议始终使用，支持 `class`。
- `<div>` / `<span>`：可结合 `t-component` 插入自定义 OWL 组件。

---

## ✅ 最佳实践建议

1. **优先使用 `attrs` 而非静态 `readonly/invisible`**：实现动态响应。
2. **合理使用 `options`**：定制 widget 行为（如颜色、尺寸）。
3. **避免过度嵌套**：保持表单结构清晰。
4. **权限控制靠后端**：XML 属性仅控制 UI，不替代安全规则。
5. **测试响应式行为**：确保 `attrs` 表达式在各种状态下正确生效。

---

如需针对特定字段类型（如 `many2many`、`date`、`selection`）的属性细节，也可进一步展开。