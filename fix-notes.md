## 修复 Gemini 2.5 报错的改动记录

背景：Raycast AI Actions 使用工具的 TypeScript 输入生成 JSON schema。`create-location` 工具原先使用 Raycast 的 `Icon` 大枚举且字段多为可选，导致 Gemini 2.5 报 “schema branching too much” 错误（GPT 系列可接受）。

改动思路：收缩最“膨胀”的字段，把地点的 `icon` 改成小型字符串枚举，并在渲染端映射回 Raycast 图标；顺带将相关输入的可选值显式化，减少分支复杂度。

具体修改：
1) `src/tools/create-location.ts`
   - `icon` 改为有限的字符串枚举 `"home" | "work" | "gym" | "store" | "school" | "other"`。
   - `proximity` 改为枚举 `"enter" | "leave"`。

2) `src/hooks/useLocations.ts`
   - 定义 `LocationIcon` 类型与 `resolveLocationIcon` 映射函数，将枚举值转成 Raycast 的具体图标；未知/旧值回退到 `Icon.Pin`。

3) `src/components/LocationForm.tsx`
   - 图标下拉只提供小枚举选项，默认值安全，提交时写入枚举值。

4) `src/manage-locations.tsx` 与 `src/components/ReminderActions.tsx`
   - 展示/选择地点时使用 `resolveLocationIcon` 映射，而不再直接存放 Raycast 大枚举。

测试：未运行自动化测试（本次为 schema/类型收敛修改）。
