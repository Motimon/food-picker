# Design Spec: 您吃了吗？（Do You Eat Yet?）

**Date:** 2026-06-03
**Status:** Approved

## Overview

A single-page mobile web app (PWA-friendly) that helps the user decide what to eat by randomly picking a nearby restaurant within a specified budget range. GPS-based location, Amap (高德地图) for POI search, no backend.

## Target Audience

Single user (the developer), zero coding experience. AI-assisted maintenance.

## Tech Stack

| Layer | Choice |
|-------|--------|
| Everything | Single HTML file (HTML + CSS + vanilla JS) |
| Location | Browser Geolocation API + Amap Geolocation fallback |
| Restaurant search | Amap Web JS API v2 (`AMap.PlaceSearch`) |
| Persistence | `localStorage` (history, last position) |
| Map display | Amap map component embedded in result card |

## API Dependencies

**Amap (高德地图) Web JS API v2:**
- `AMap.Geolocation` — GPS + IP fallback positioning, reverse geocoding
- `AMap.PlaceSearch` — nearby POI search with `keywords: "餐饮"`, radius filter
- Free tier: ~5000 calls/day, sufficient for single user

**API Key required:** User must register at https://lbs.amap.com, create a "Web 服务" application, and paste the Key.

## User Flow

```
[Open page]
    ↓
[Browser requests GPS permission]
    ↓ (auto-fallback: Amap Geolocation)
[Display: "当前位置: XX区XX路"]
    ↓
[User sets: search radius + budget range]
    ↓
[Tap "帮我决定"]
    ↓
[Amap PlaceSearch → filter by budget → random pick]
    ↓
[Show result card: name, rating, avg cost, distance, mini map]
    ↓
[User taps "换一个" → re-random, exclude seen]
[User taps "就它了" → save to history, confirm state]
```

## UI Layout (Single Screen, No Scroll)

```
┌──────────────────────────────────┐
│        🍜 您吃了吗？              │
│     Do You Eat Yet?              │  ← fixed header
├──────────────────────────────────┤
│ 📍 当前位置: XX区XX路  [重定位]   │
│                                  │
│ 搜索范围:  [500m ▼]              │  ← dropdown
│                                  │
│ 人均预算:  [¥20 ═══╪══ ¥80]     │  ← dual range slider or two inputs
│                                  │
│ ┌──────────────────────────────────┐
│ │         🎲 帮我决定              │  ← prominent CTA button
│ └──────────────────────────────────┘
│                                  │
│ ┌──────────────────────────────────┐
│ │ 🍕 餐厅名            ★4.2      │  ← result card (hidden initially)
│ │ 人均 ¥68/人         320m       │
│ │ 地址: XX路XX号                  │
│ │ [换一个] [导航去] [就它了]      │
│ └──────────────────────────────────┘
│                                  │
│ 历史: 🍜老王麻辣烫 🍔麦当劳 ...    │  ← recent history row
└──────────────────────────────────┘
```

## Data Model

```
Restaurant (from Amap POI):
  - id: string (POI ID)
  - name: string (店名)
  - address: string (地址)
  - location: {lng, lat}
  - distance: number (meters)
  - rating: number (1-5, optional)
  - avgCost: number (人均消费, optional, from deep_info/biz_ext)

SearchParams:
  - center: {lng, lat}
  - radius: number (meters)
  - minBudget: number (CNY)
  - maxBudget: number (CNY)

History (localStorage):
  - items: Array<{name, date, avgCost}>
```

## Edge Cases

| Scenario | Handling |
|----------|----------|
| GPS permission denied | Show manual address input fallback |
| No restaurants in radius | Prompt to expand radius |
| No results in budget range | Prompt to adjust budget |
| Amap API fails | Show error + retry button |
| No API Key configured | Red warning banner at top |
| Browser lacks Geolocation | Fallback to Amap IP-based positioning |
| Page opened on desktop | Mobile-first layout works at narrow width; center column on wide screens |

## Color Palette

Warm, food-appetite colors:
- Primary: Orange (#FF6B35)
- Background: Warm cream (#FFF8F0)
- Card: White (#FFFFFF)
- Text: Dark gray (#333333)
- Accent: Green (#4CAF50) for confirm action

## Non-Goals (v1)

- User accounts / login
- Backend server / database
- Google Play / App Store publication
- Multi-language support
- Dietary preference filters (vegetarian, halal, etc.)
- Real-time restaurant availability or delivery status
