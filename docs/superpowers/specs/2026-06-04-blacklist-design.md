# Design Spec: 黑名单功能

**Date:** 2026-06-04
**Status:** Approved

## Overview

Add a blacklist feature to the food picker app. Users can blacklist restaurants they don't want to see again. Blacklisted restaurants are permanently excluded from search results.

## Data Model

```
Blacklist (localStorage key: food_picker_blacklist):
  - items: Array<{name: string, date: ISO string}>
```

Max 50 items. Same structure as history for consistency.

## UI Changes

### Result Card
- Add "👎 拉黑" button next to existing action buttons (4th button)

### History Section Area
- Add "黑名单" button next to "📋 最近选择" and "清空" button

### Blacklist Management Modal
- Full-screen overlay popup triggered by "黑名单" button
- Header: "🚫 黑名单" with "[编辑]" toggle button
- List: each item shows name + date (formatted M/D)
- Normal mode: items displayed as read-only list
- Edit mode: checkbox appears before each item, footer shows action buttons
- Footer buttons (edit mode): "删除选中" + "清除全部"
- Close button always visible

## Filter Logic

After `searchNearby` returns results, filter out any restaurant whose `name` matches an entry in the blacklist. This happens BEFORE budget filtering.

```javascript
function excludeBlacklisted(restaurants) {
  const blacklist = loadBlacklist();
  const names = blacklist.map(item => item.name);
  return restaurants.filter(r => names.indexOf(r.name) === -1);
}
```

## User Interactions

| Action | Behavior |
|--------|----------|
| Tap "👎 拉黑" on result | Save to blacklist, show "已拉黑 ✓" briefly, auto-switch to next restaurant |
| Tap "黑名单" button | Open management modal |
| Tap "编辑" in modal | Enter batch-select mode with checkboxes |
| Tap "删除选中" | Remove checked items, refresh list |
| Tap "清除全部" | Confirm dialog → clear all → refresh list |
| Tap "关闭"/backdrop | Close modal |

## Edge Cases

| Scenario | Handling |
|----------|----------|
| Blacklist empty | Modal shows "还没有拉黑任何店铺" |
| All results blacklisted | Show "该预算区间内的餐厅已经浏览完了哦~" (same as existing) |
| Duplicate blacklist entry | Prevent adding same name twice |
| Max 50 reached | Alert "黑名单已满，请先清理" |
| "就它了" on already blacklisted | Not possible (blacklisted items excluded from results) |

## Non-Goals (v1)

- Blacklisting food categories (only store names)
- Sync across devices
- Undo blacklist entry from result card
