在 Odoo 中，`<record>` 是用于向数据库模型（Model）**插入或更新记录**的核心 XML 元素。虽然 `<record>` 标签本身只有两个直接属性（`id` 和 `model`），但其功能通过 `<field>` 子标签和上下文机制实现强大配置能力。

下面将：

1. **详解 `<record>` 所有可操作属性与用法**  
2. **提供一个包含所有用法的完整 XML 示例（带注释）**  
3. **总结所有属性**  
4. **对比 `ir.ui.view`、`res.partner`、`ir.actions.act_window` 三大模型的区别**  
5. **为每个模型提供独立 Demo**

---

## ✅ 一、完整示例：包含 `<record>` 所有可能用法的 XML（带详细注释）

```xml
<odoo>

    <!-- 
        =============================
        1. 创建一个 ir.ui.view 记录（视图定义）
        =============================
    -->
    <record id="view_custom_partner_form" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.custom</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/> <!-- 继承官方表单 -->
        <field name="priority" eval="20"/> <!-- 数字值用 eval 更安全 -->
        <field name="active" eval="True"/> <!-- 布尔值必须用 eval -->
        <field name="arch" type="xml">
            <!-- type="xml" 表示子内容作为原始 XML 保存 -->
            <xpath expr="//field[@name='name']" position="after">
                <field name="custom_field"/>
            </xpath>
        </field>
    </record>

    <!-- 
        =============================
        2. 创建一个 res.partner 记录（业务数据）
        =============================
    -->
    <record id="demo_partner_acme" model="res.partner">
        <field name="name">ACME Corporation</field>
        <field name="email">contact@acme.com</field>
        <field name="phone">+1-555-1234</field>
        <field name="customer_rank" eval="1"/> <!-- 客户等级 -->
        <field name="supplier_rank" eval="0"/>
        <field name="company_type">company</field>
        <field name="is_company" eval="True"/>
        <!-- Many2one 字段：引用国家 -->
        <field name="country_id" ref="base.us"/> <!-- ref 引用 base 模块的美国记录 -->
        <!-- Boolean 字段 -->
        <field name="active" eval="True"/>
        <!-- Datetime 字段（使用 eval + time） -->
        <field name="create_date" eval="time.strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>

    <!-- 
        =============================
        3. 创建一个 ir.actions.act_window 记录（窗口动作）
        =============================
    -->
    <record id="action_open_custom_partners" model="ir.actions.act_window">
        <field name="name">Custom Partners</field>
        <field name="res_model">res.partner</field>
        <field name="view_mode">tree,form</field>
        <!-- view_ids：指定多个视图（list of (view_id, view_mode)） -->
        <field name="view_ids" eval="[(5, 0, 0), 
                                      (0, 0, {'view_mode': 'tree', 'view_id': ref('view_partner_tree')}),
                                      (0, 0, {'view_mode': 'form', 'view_id': ref('view_partner_form')})]"/>
        <!-- domain：过滤条件 -->
        <field name="domain" eval="[('customer_rank', '>', 0)]"/>
        <!-- context：默认上下文 -->
        <field name="context" eval="{'search_default_customer': 1}"/>
        <!-- help：帮助文本（HTML） -->
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new customer.
            </p>
        </field>
        <!-- limit：每页记录数 -->
        <field name="limit" eval="80"/>
        <!-- target：打开方式（current/new） -->
        <field name="target">current</field>
    </record>

    <!-- 
        =============================
        4. 高级用法：noupdate 控制升级行为
        =============================
    -->
    <data noupdate="1">
        <record id="config_param_demo" model="ir.config_parameter">
            <field name="key">my_module.demo_enabled</field>
            <field name="value">True</field>
        </record>
    </data>

</odoo>
```

> 💡 **关键说明**：
> - `<record>` 只有 `id` 和 `model` 两个属性。
> - 所有字段赋值通过 `<field>` 实现，支持：普通值、`ref`、`eval`、`search`、`type="xml/html"`。
> - `noupdate="1"` 确保配置数据在模块升级时不被覆盖。

---

## 📚 二、`<record>` 所有可操作属性总结

### 1. `<record>` 自身属性（仅 2 个）

| 属性 | 必填 | 说明 |
|------|------|------|
| `id` | ✅ 是 | **XML ID（External ID）**，模块内唯一标识，用于引用（如 `ref="id"`） |
| `model` | ✅ 是 | **目标 Odoo 模型名称**，如 `ir.ui.view`、`res.partner` |

> ❌ **不存在的“属性”**：  
> `string`, `class`, `domain`, `context` 等都是**模型字段名**，不是 `<record>` 的属性！

---

### 2. `<field>` 支持的赋值方式（扩展 `<record>` 能力）

| 方式 | 语法 | 适用场景 |
|------|------|----------|
| **普通值** | `<field name="x">text</field>` | `Char`, `Text`, `Selection` |
| **引用（ref）** | `<field name="x" ref="other_id"/>` | `Many2one`, `Reference` |
| **表达式（eval）** | `<field name="x" eval="python_code"/>` | `Dict`, `List`, `Bool`, `Int`, 动态值 |
| **动态查找（search）** | `<field name="x" search="[('f','=',v)]"/>` | `Many2one`（Odoo 14+） |
| **XML 结构** | `<field name="arch" type="xml">...</field>` | 视图 `arch` 字段 |
| **HTML 内容** | `<field name="help" type="html">...</field>` | 富文本字段 |

---

### 3. 上下文控制（非 `<record>` 属性，但影响行为）

| 机制 | 作用 |
|------|------|
| `<data noupdate="1">` | 模块升级时跳过该记录（用于配置、用户数据） |
| 相同 `id` | 覆盖/更新已有记录（继承机制） |

---

## 🔍 三、三大模型区别详解

| 模型 | 类型 | 用途 | 是否业务数据 | 是否 UI 配置 | 典型字段 |
|------|------|------|---------------|----------------|--------|
| **`ir.ui.view`** | 技术模型 | 定义视图结构（form/tree/kanban等） | ❌ 否 | ✅ 是 | `name`, `model`, `arch`, `type`, `inherit_id` |
| **`res.partner`** | 业务模型 | 存储客户/供应商/联系人信息 | ✅ 是 | ❌ 否 | `name`, `email`, `phone`, `customer_rank` |
| **`ir.actions.act_window`** | 技术模型 | 定义点击菜单后打开的窗口行为 | ❌ 否 | ✅ 是 | `name`, `res_model`, `view_mode`, `domain`, `context` |

### 🎯 核心区别总结：

| 维度 | `ir.ui.view` | `res.partner` | `ir.actions.act_window` |
|------|--------------|----------------|--------------------------|
| **目的** | “怎么显示” | “存什么数据” | “点开后做什么” |
| **用户可见** | 不可见（后台配置） | 可见（业务记录） | 不可见（行为定义） |
| **依赖关系** | 被 Action 引用 | 被其他业务模型引用 | 被 Menu 引用 |
| **是否可编辑** | 开发者通过 XML 配置 | 用户通过 UI 编辑 | 开发者配置，用户不可见 |

---

## 🧪 四、各自独立 Demo

### Demo 1：`ir.ui.view` —— 自定义表单视图

```xml
<record id="view_partner_form_add_custom" model="ir.ui.view">
    <field name="name">res.partner.form.add.custom.field</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
        <xpath expr="//field[@name='website']" position="after">
            <field name="custom_field" placeholder="Enter custom info..."/>
        </xpath>
    </field>
</record>
```

> ✅ 效果：在联系人表单中 `Website` 字段后添加一个自定义字段。

---

### Demo 2：`res.partner` —— 创建演示客户

```xml
<record id="demo_partner_xyz" model="res.partner">
    <field name="name">XYZ Tech Ltd.</field>
    <field name="email">info@xyztech.com</field>
    <field name="phone">+44-20-1234-5678</field>
    <field name="customer_rank" eval="1"/>
    <field name="is_company" eval="True"/>
    <field name="country_id" ref="base.uk"/>
</record>
```

> ✅ 效果：在数据库中创建一条客户记录，用户可在“联系人”应用中看到。

---

### Demo 3：`ir.actions.act_window` —— 自定义动作

```xml
<record id="action_partners_customers_only" model="ir.actions.act_window">
    <field name="name">My Customers</field>
    <field name="res_model">res.partner</field>
    <field name="view_mode">tree,form</field>
    <field name="domain" eval="[('customer_rank', '>', 0)]"/>
    <field name="context" eval="{'default_customer_rank': 1}"/>
    <field name="help" type="html">
        <p>Add your first customer!</p>
    </field>
</record>
```

> ✅ 效果：创建一个只显示客户的列表动作，常用于菜单绑定。

---

## ✅ 最佳实践建议

1. **视图（`ir.ui.view`）**：优先使用 `inherit_id` 继承，避免重写整个视图。
2. **业务数据（`res.partner`）**：放在 `demo/` 目录，用 `noupdate="0"`。
3. **动作（`ir.actions.act_window`）**：配合菜单使用，`domain` 和 `context` 是核心。
4. **统一命名**：`id` 使用 `model_description` 格式（如 `action_partner_customer`）。

---

通过 `<record>`，你可以声明式地构建 Odoo 应用的**UI 配置**（视图、动作）和**业务数据**（客户、产品等）。理解这三类模型的区别，是掌握 Odoo 模块开发的关键！