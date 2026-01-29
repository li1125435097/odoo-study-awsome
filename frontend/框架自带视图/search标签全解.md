在 **Odoo 17** 中，`<search>` 视图（搜索视图）**不是独立显示的视图**，而是为其他主视图（如 `list`、`kanban`、`graph`、`pivot` 等）提供**搜索栏（Search Bar）功能**的核心组件。它定义了用户可使用的：

- 搜索字段（Searchable Fields）  
- 过滤器（Filters）  
- 分组选项（Group By）  
- 扩展筛选器（Advanced Filters）  
- 默认搜索/分组行为  

`<search>` 视图通过 `ir.ui.view` 注册，系统会自动将其注入到关联模型的所有支持搜索的视图顶部。

---

## ✅ 完整示例：包含所有可用元素和属性的 Search 视图 XML（带详细注释）

```xml
<!-- 文件：views/res_partner_search_view.xml -->
<odoo>
    <record id="view_res_partner_filter" model="ir.ui.view">
        <field name="name">res.partner.search</field>
        <field name="model">res.partner</field>
        <field name="arch" type="xml">
            <!-- 
                <search> 根元素本身不支持属性（如 class, js_class 等无效）
                所有功能通过子元素实现：
                  - <field>: 声明可搜索字段
                  - <filter>: 预设过滤器或分组
                  - <separator>: 视觉分隔（仅在高级搜索中可见）
                  - <group>: 逻辑分组（用于“扩展筛选”面板）
            -->
            <search string="Partners">
                <!-- 
                    1. 可搜索字段（出现在搜索框下拉建议中）
                    - name: 字段名（必须存在于模型中）
                    - string: 显示名称（可选，默认取字段定义的 string）
                    - filter_domain: 自定义搜索逻辑（高级用法）
                    - operator: 搜索操作符（默认 'ilike'）
                -->
                <field name="name" string="Name"/>
                <field name="email" string="Email"/>
                <field name="phone" string="Phone"/>
                
                <!-- 
                    高级：自定义搜索域（例如模糊匹配多个字段）
                    当用户在搜索框输入时，触发此 domain
                -->
                <field name="category_id" 
                       string="Tag"
                       filter_domain="[('category_id.name', 'ilike', self)]"/>

                <!-- 
                    2. 预设过滤器（Filter Buttons）
                    - name: 唯一标识（用于 context 或代码引用）
                    - string: 按钮显示文本
                    - domain: 过滤条件（列表格式）
                    - help: 悬停提示（可选）
                    - icon: 图标（企业版支持，如 'fa fa-star'）
                -->
                <filter name="customers"
                        string="Customers"
                        domain="[('customer_rank', '>', 0)]"
                        help="Partners with customer rank > 0"/>
                
                <filter name="suppliers"
                        string="Vendors"
                        domain="[('supplier_rank', '>', 0)]"/>

                <!-- 
                    3. 默认启用的过滤器（auto=true）
                    页面加载时自动激活
                -->
                <filter name="active"
                        string="Active"
                        domain="[('active', '=', True)]"
                        default_focus="1"/> <!-- 聚焦到此过滤器（罕见用法） -->

                <!-- 
                    4. 分组选项（Group By）
                    放在 <group> 内，string 为分组菜单标题
                    - context: 必须包含 {'group_by': '字段名'}
                    - help: 描述
                -->
                <group expand="0" string="Group By">
                    <!-- expand="0" 表示默认折叠；"1" 表示展开 -->
                    <filter name="group_by_country"
                            string="Country"
                            context="{'group_by': 'country_id'}"/>
                    
                    <filter name="group_by_industry"
                            string="Industry"
                            context="{'group_by': 'industry_id'}"/>
                    
                    <filter name="group_by_salesperson"
                            string="Salesperson"
                            context="{'group_by': 'user_id'}"/>
                </group>

                <!-- 
                    5. 高级筛选分组（在“扩展筛选”面板中显示）
                    使用 <group> 包裹，但不设置 context（即非分组用途）
                -->
                <group string="Contact Details">
                    <field name="function"/>
                    <field name="title"/>
                </group>

                <!-- 
                    6. 分隔符（仅在“扩展筛选”中可见）
                -->
                <separator/>

                <!-- 
                    7. 全局默认行为（通过 <searchpanel> 以外的方式控制）
                    注意：<searchpanel> 是独立组件（用于侧边栏筛选），不属于 <search> 视图
                -->

            </search>
        </field>
    </record>
</odoo>
```

> 💡 **关键说明**：
> - `<search>` **没有根属性**（如 `class`, `js_class` 等均无效）。
> - 所有功能通过子元素 `<field>`, `<filter>`, `<group>`, `<separator>` 实现。
> - `domain` 和 `context` 使用 **Python 列表/字典语法**（字符串形式）。
> - 分组选项必须放在 `<group string="Group By">` 内，且 `expand="0"` 控制默认折叠状态。

---

## 📚 Search 视图所有可操作元素与属性总结

### 一、`<search>` 根元素
| 属性 | 说明 |
|------|------|
| （无有效属性） | `<search>` 本身不支持任何 XML 属性（如 `class`, `string` 仅作注释用） |

---

### 二、`<field>` 元素（声明可搜索字段）

| 属性 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `name` | ✅ 是 | — | 模型字段名 |
| `string` | ❌ 否 | 字段定义的 `string` | 搜索下拉中显示的标签 |
| `filter_domain` | ❌ 否 | `[('name', 'ilike', self)]` | 自定义搜索逻辑（`self` 代表用户输入值） |
| `operator` | ❌ 否 | `'ilike'` | 搜索操作符（如 `'='`, `'!='`, `'>'` 等） |

> 🔍 **`filter_domain` 示例**：
> ```xml
> <!-- 搜索客户名称或公司名称 -->
> <field name="name" filter_domain="['|', ('name', 'ilike', self), ('company_name', 'ilike', self)]"/>
> ```

---

### 三、`<filter>` 元素（预设过滤器或分组）

| 属性 | 必填 | 说明 |
|------|------|------|
| `name` | ✅ 是 | 唯一标识（用于代码或 URL 引用） |
| `string` | ✅ 是 | 按钮显示文本 |
| `domain` | ❌ 否 | 过滤条件（如 `[('state', '=', 'done')]`） |
| `context` | ❌ 否 | 上下文变更（如 `{'group_by': 'stage_id'}`） |
| `help` | ❌ 否 | 悬停提示文本 |
| `icon` | ❌ 否 | 图标类（企业版支持，如 `'fa fa-user'`） |
| `default_focus` | ❌ 否 | 是否默认聚焦（极少使用） |

> ⚠️ **注意**：
> - 一个 `<filter>` **不能同时有 `domain` 和 `context`**（Odoo 会忽略其中一个）。
> - 若只有 `context`，则作为 **分组按钮**；若只有 `domain`，则作为 **过滤按钮**。

---

### 四、`<group>` 元素（逻辑分组）

| 属性 | 说明 |
|------|------|
| `string` | 分组标题（在“扩展筛选”或“Group By”菜单中显示） |
| `expand` | `"0"`（默认折叠）或 `"1"`（默认展开）——**仅对“Group By”分组有效** |

> 📌 **两种用途**：
> 1. **分组选项容器**：`<group string="Group By">` + `<filter context="{'group_by': ...}"/>`
> 2. **高级筛选分组**：`<group string="Contact Info">` + `<field name="..."/>`

---

### 五、`<separator>` 元素
| 说明 |
|------|
| 在“扩展筛选”面板中插入一条水平分隔线（视觉分隔） |

---

### 六、特殊行为控制

| 功能 | 实现方式 |
|------|----------|
| **默认过滤器** | 设置 `<filter domain="..."/>` 并配合 `ir.actions.act_window` 的 `context`（如 `{'search_default_customers': 1}`） |
| **默认分组** | 在 Action 的 `context` 中设置 `{'group_by': ['field1', 'field2']}` |
| **搜索建议排序** | 字段声明顺序决定下拉建议优先级 |

> ✅ **在 Action 中激活默认过滤器示例**：
> ```xml
> <field name="context">{'search_default_customers': 1, 'group_by': ['country_id']}</field>
> ```

---

### 七、不支持的功能（常见误区）

| 试图使用的属性/元素 | 实际效果 |
|---------------------|----------|
| `<search class="...">` | 忽略 |
| `<field invisible="1">` | 无效（`invisible` 不适用于 search 视图） |
| `<button>` | 不允许（search 视图只接受特定子元素） |
| `attrs`、`options` | 无效 |

---

## ✅ 最佳实践建议

1. **合理选择可搜索字段**：只暴露业务相关的字段，避免性能问题。
2. **使用 `filter_domain` 增强搜索**：支持跨字段模糊匹配。
3. **命名清晰的过滤器**：`string` 应直观（如 “My Leads” 而非 “mine”）。
4. **分组默认折叠**：`expand="0"` 避免界面杂乱。
5. **通过 Action 控制默认行为**：而不是硬编码在 search 视图中。
6. **避免过多过滤器**：保持搜索栏简洁，复杂条件放入“扩展筛选”。

---

## 🧩 典型应用场景

### 场景 1：销售机会搜索
```xml
<search>
    <field name="name"/>
    <field name="partner_id"/>
    <filter name="my_leads" string="My Leads" domain="[('user_id', '=', uid)]"/>
    <group string="Group By">
        <filter name="by_stage" string="Stage" context="{'group_by': 'stage_id'}"/>
    </group>
</search>
```

### 场景 2：任务管理
```xml
<search>
    <field name="name"/>
    <filter name="today" string="Today" domain="[('date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
    <filter name="overdue" string="Overdue" domain="[('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d')), ('state', '!=', 'done')]"/>
</search>
```

---

通过以上配置，你可以构建强大而用户友好的搜索体验。记住：**`<search>` 视图是 Odoo 数据探索的入口**，设计良好的搜索能极大提升用户效率。