# Ant Design Sub-component Cheat Sheet (Next.js App Router)

| Component   | Sub-components (❌ needs `"use client";`) | ✅ Server-safe Alternative |
|-------------|-------------------------------------------|-----------------------------|
| **Select**  | `Select.Option`, `Select.OptGroup`       | Use `options={[...]}` prop |
| **DatePicker** | `DatePicker.RangePicker`, `MonthPicker`, `WeekPicker`, `QuarterPicker` | Use `picker="date" | month | week | quarter"` |
| **TimePicker** | `TimePicker.RangePicker`              | Use `picker="time"` |
| **Form**    | `Form.Item`, `Form.List`, `Form.Provider` | Use plain `<form>` + AntD inputs, or custom wrapper |
| **Upload**  | `Upload.Dragger`                         | Use `<Upload listType="picture-card" />` or props |
| **Tabs**    | `Tabs.TabPane`                           | Use `items={[{ label, key, children }]}` prop |
| **Table**   | `Table.Column`, `Table.ColumnGroup`, `Table.Summary` | Use `columns={[...]}` prop |
| **Image**   | `Image.PreviewGroup`                     | Use `preview={{ groupName: 'group' }}` prop |
| **Descriptions** | `Descriptions.Item`                | Use `items={[...]}` prop |
| **List**    | `List.Item`, `List.Item.Meta`            | Use `renderItem` prop |
| **Layout**  | `Layout.Header`, `Layout.Footer`, `Layout.Sider`, `Layout.Content` | Use `div` with AntD `Layout` props/styles |
| **Menu**    | `Menu.Item`, `Menu.SubMenu`, `Menu.ItemGroup` | Use `items={[...]}` prop |
| **Dropdown**| `Dropdown.Button`                        | Use `<Dropdown menu={{ items }} />` |
| **Card**    | `Card.Grid`, `Card.Meta`                 | Use props (`cover`, `actions`, `title`, `extra`) |
| **Steps**   | `Steps.Step`                             | Use `items={[...]}` prop |
| **Collapse**| `Collapse.Panel`                         | Use `items={[...]}` prop |
| **Skeleton**| `Skeleton.Input`, `Skeleton.Button`, `Skeleton.Avatar`, `Skeleton.Image` | Use `Skeleton` with `avatar`, `paragraph`, `title` props |
| **PageHeader** (deprecated v5) | `PageHeader.Back`     | Use `Button` or `ArrowLeftOutlined` directly |

---

## 🎯 Rule of Thumb
- If you write `<Component.SubComponent> ... </Component.SubComponent>` → **must use `"use client";`**.  
- If you use `items`, `options`, or `columns` props → **safe as Server Component**.
