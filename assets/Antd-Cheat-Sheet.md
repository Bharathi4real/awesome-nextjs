# Complete Ant Design Sub-component Cheat Sheet (Next.js App Router)

## 🎯 Rule of Thumb
- If you write `<Component.SubComponent> ... </Component.SubComponent>` → **must use `"use client";`**.  
- If you use `items`, `options`, or `columns` props → **safe as Server Component**.

---

## 📊 Data Display Components

| Component   | Sub-components (needs `"use client";`) |  Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Table**   | `Table.Column`, `Table.ColumnGroup`, `Table.Summary`, `Table.SELECTION_COLUMN`, `Table.EXPAND_COLUMN` | Use `columns={[...]}` prop with `summary` function |
| **Descriptions** | `Descriptions.Item`                | Use `items={[{ key, label, children }]}` prop |
| **List**    | `List.Item`, `List.Item.Meta`            | Use `renderItem={(item) => <List.Item>...</List.Item>}` prop |
| **Card**    | `Card.Grid`, `Card.Meta`                 | Use props (`cover`, `actions`, `title`, `extra`, `bodyStyle`) |
| **Statistic** | `Statistic.Countdown`                  | Use separate `<Statistic />` with `value` and `formatter` |
| **Image**   | `Image.PreviewGroup`                     | Use `preview={{ visible, onVisibleChange }}` prop |
| **Calendar** | `Calendar.Header`                       | Use `headerRender` prop |
| **Tree**    | `Tree.TreeNode`, `Tree.DirectoryTree`    | Use `treeData={[...]}` prop |
| **Timeline** | `Timeline.Item`                         | Use `items={[{ children, dot, color }]}` prop |

---

## 📝 Data Entry Components

| Component   | Sub-components (needs `"use client";`) |  Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Form**    | `Form.Item`, `Form.List`, `Form.Provider`, `Form.ErrorList` | Use plain `<form>` + AntD inputs, or custom wrapper |
| **Select**  | `Select.Option`, `Select.OptGroup`       | Use `options={[{ value, label, disabled }]}` prop |
| **DatePicker** | `DatePicker.RangePicker`, `DatePicker.MonthPicker`, `DatePicker.WeekPicker`, `DatePicker.QuarterPicker`, `DatePicker.YearPicker`, `DatePicker.TimePicker` | Use `picker="date\|month\|week\|quarter\|year\|time"` |
| **TimePicker** | `TimePicker.RangePicker`              | Use `<DatePicker picker="time" />` |
| **Upload**  | `Upload.Dragger`                         | Use `<Upload listType="picture-card" />` or `dragger={true}` |
| **TreeSelect** | `TreeSelect.TreeNode`                 | Use `treeData={[...]}` prop |
| **Cascader** | `Cascader.Panel`                        | Use `options={[...]}` with nested structure |
| **Transfer** | `Transfer.List`, `Transfer.Operation`, `Transfer.Search` | Use built-in props (`render`, `operations`, `showSearch`) |
| **Mentions** | `Mentions.Option`                       | Use `options={[...]}` prop |
| **AutoComplete** | `AutoComplete.Option`, `AutoComplete.OptGroup` | Use `options={[...]}` prop |

---

## 🎨 Layout & Navigation Components

| Component   | Sub-components (needs `"use client";`) |  Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Layout**  | `Layout.Header`, `Layout.Footer`, `Layout.Sider`, `Layout.Content` | Use individual components or styled `div` |
| **Menu**    | `Menu.Item`, `Menu.SubMenu`, `Menu.ItemGroup`, `Menu.Divider` | Use `items={[{ key, label, children, type }]}` prop |
| **Breadcrumb** | `Breadcrumb.Item`, `Breadcrumb.Separator` | Use `items={[{ title, href }]}` prop |
| **Steps**   | `Steps.Step`                             | Use `items={[{ title, description, icon }]}` prop |
| **Tabs**    | `Tabs.TabPane`                           | Use `items={[{ label, key, children }]}` prop |
| **Dropdown**| `Dropdown.Button`                        | Use `<Dropdown menu={{ items }} />` |
| **Pagination** | N/A (no sub-components)               | ✅ Server-safe by default |
| **Affix**   | N/A (no sub-components)                  | ✅ Server-safe by default |

---

## 💬 Feedback Components

| Component   | Sub-components (needs `"use client";`) |  Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Alert**   | `Alert.ErrorBoundary`                    | Use `<Alert />` with `type` and `message` props |
| **Message** | N/A (programmatic API)                   | ⚠️ Client-side only (`message.success()`) |
| **Notification** | N/A (programmatic API)              | ⚠️ Client-side only (`notification.open()`) |
| **Modal**   | `Modal.confirm`, `Modal.info`, `Modal.success`, `Modal.error`, `Modal.warning` | Use `<Modal />` component with state |
| **Drawer**  | N/A (no sub-components)                  | ✅ Server-safe with state management |
| **Progress** | `Progress.Line`, `Progress.Circle`      | Use `type="line\|circle\|dashboard"` prop |
| **Popconfirm** | N/A (no sub-components)               | ✅ Server-safe by default |
| **Popover** | N/A (no sub-components)                  | ✅ Server-safe by default |
| **Tooltip** | N/A (no sub-components)                  | ✅ Server-safe by default |
| **Result**  | N/A (no sub-components)                  | ✅ Server-safe by default |
| **Skeleton** | `Skeleton.Input`, `Skeleton.Button`, `Skeleton.Avatar`, `Skeleton.Image`, `Skeleton.Node` | Use `<Skeleton />` with `avatar`, `paragraph`, `title`, `active` props |
| **Spin**    | N/A (no sub-components)                  | ✅ Server-safe by default |

---

## 🔧 General Components

| Component   | Sub-components (needs `"use client";`) |  Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Button**  | `Button.Group`                           | Use `<Space.Compact>` or custom CSS Grid/Flex |
| **Icon**    | All individual icons (e.g., `PlusOutlined`) | ✅ Server-safe by default |
| **Typography** | `Typography.Text`, `Typography.Title`, `Typography.Paragraph`, `Typography.Link` | Use individual components or HTML tags |
| **Divider** | N/A (no sub-components)                  | ✅ Server-safe by default |
| **Space**   | `Space.Compact`                          | Use `<Space />` with `compact` prop |
| **Anchor**  | `Anchor.Link`                            | Use `items={[{ key, href, title }]}` prop |
| **BackTop** | N/A (no sub-components)                  | ✅ Server-safe by default |
| **ConfigProvider** | N/A (no sub-components)           | ✅ Server-safe by default |
| **Empty**   | N/A (no sub-components)                  | ✅ Server-safe by default |
| **Collapse** | `Collapse.Panel`                        | Use `items={[{ key, label, children }]}` prop |

---

## 🎪 Advanced/Specialized Components

| Component   | Sub-components (needs `"use client";`) |  Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Tour**    | N/A (no sub-components)                  | ✅ Server-safe with proper state |
| **FloatButton** | `FloatButton.Group`, `FloatButton.BackTop` | Use individual `<FloatButton />` with grouping |
| **Watermark** | N/A (no sub-components)                | ✅ Server-safe by default |
| **QRCode**  | N/A (no sub-components)                  | ✅ Server-safe by default |
| **Segmented** | N/A (no sub-components)                | ✅ Server-safe with `options={[...]}` |

---

## 📚 Typography Components Deep Dive

```tsx
// ❌ Client Component Required
"use client";
<Typography.Title level={1}>Title</Typography.Title>
<Typography.Text strong>Bold text</Typography.Text>
<Typography.Paragraph>Content</Typography.Paragraph>
<Typography.Link href="/link">Link</Typography.Link>

// ✅ Server Component Safe
import { Typography } from 'antd';
const { Title, Text, Paragraph, Link } = Typography;

<Title level={1}>Title</Title>
<Text strong>Bold text</Text>
<Paragraph>Content</Paragraph>
<Link href="/link">Link</Link>
```

---

## 🎯 Common Patterns

### Form Pattern
```tsx
// ❌ Client Component Required
<Form>
  <Form.Item name="username">
    <Input />
  </Form.Item>
</Form>

// ✅ Server Component Safe (with client wrapper)
<FormWrapper>
  <Input name="username" />
</FormWrapper>
```

### Menu Pattern
```tsx
// ❌ Client Component Required
<Menu>
  <Menu.Item key="1">Item 1</Menu.Item>
  <Menu.SubMenu title="Submenu">
    <Menu.Item key="2">Item 2</Menu.Item>
  </Menu.SubMenu>
</Menu>

// ✅ Server Component Safe
<Menu items={[
  { key: '1', label: 'Item 1' },
  { key: 'sub1', label: 'Submenu', children: [
    { key: '2', label: 'Item 2' }
  ]}
]} />
```

### Table Pattern
```tsx
// ❌ Client Component Required
<Table dataSource={data}>
  <Table.Column title="Name" dataIndex="name" />
  <Table.Column title="Age" dataIndex="age" />
</Table>

// ✅ Server Component Safe
<Table 
  dataSource={data}
  columns={[
    { title: 'Name', dataIndex: 'name' },
    { title: 'Age', dataIndex: 'age' }
  ]}
/>
```

---

## 🚨 Special Cases & Notes

### Always Client-Side Components
- `message.*` (programmatic API)
- `notification.*` (programmatic API)  
- `Modal.confirm` and similar static methods
- Any component using `ref` callbacks
- Components with event handlers (onClick, onChange, etc.)

### Hybrid Approach
For complex applications, consider:
1. Server components for static content
2. Client components for interactive parts
3. Proper component boundary separation

### Migration Tips
1. Replace sub-components with prop-based alternatives
2. Extract interactive logic to separate client components
3. Use composition instead of inheritance patterns
4. Leverage Next.js 13+ app directory structure

---

## Contribution
Want to suggest additions or corrections? Open a pull request!
