---
title: 技术探索
date: 2025-04-21
updated: 2025-04-21
categories: 技术分享
tags:
  - 用户体验
  - 笔记
  - 用户体验
top: 1
---

## 提升博客温度的第一步，从「看见」访客开始

“当虚拟空间的连接有了地理坐标，数字背后的个体便有了更真实的轮廓。”—— 这是我开发动态公告栏的初衷。通过技术手段将抽象的 IP 地址转化为具象的城市坐标，再用距离和问候语搭建起微妙的情感桥梁，这个过程充满了理性与温度的碰撞。

## 一、核心功能解析

### 1. 精准定位：让 IP 说出「你从哪里来」

通过 VORE-API 接口获取用户 IP 对应的中文地址，无需用户授权即可实现城市级定位，数据结构清晰适配：

```json
{  
  "ipdata": {  
    "info1": "河南省",       // 省份  
    "info2": "郑州市",       // 城市  
    "info3": "巩义"          // 区县  
  }  
}  
```

### 2. 距离计算：丈量虚拟与现实的尺度

结合 ip-api.com 提供的经纬度数据，运用球面三角公式计算用户与站长的直线距离：
```typescript
function calculateDistance(loc1, loc2) {  
  const R = 6371; // 地球半径（公里）  
  const dLat = (loc2.lat - loc1.lat) * Math.PI / 180;  
  const dLon = (loc2.lng - loc1.lng) * Math.PI / 180;  
  const a = Math.sin(dLat/2) * Math.sin(dLat/2) +  
            Math.cos(loc1.lat*Math.PI/180) * Math.cos(loc2.lat*Math.PI/180) *  
            Math.sin(dLon/2) * Math.sin(dLon/2);  
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));  
}  
```

// 示例：用户与站长同处郑州时，距离显示为 0.00 公里

### 3. 智能问候：用时间编织情感纽带

根据当前小时数自动切换问候语，让每个访问时刻都有专属温度：
```typescript
const hour = new Date().getHours();  
greeting.value = hour < 12 ? '早上好，代码与咖啡缺一不可☕' :  
                  hour < 18 ? '下午好，此刻适合攻克技术难题💻' :  
                  '晚上好，记得抬头看看星空✨';  
```

## 二、技术实现与挑战

### 1. TypeScript 类型安全实践

定义接口规范数据结构，避免运行时字段缺失错误：
```typescript
interface IPLocation {  
  province: string;  
  city: string;  
  district?: string; // 可选区县字段  
  coordinates: { lat: number; lng: number };  
}  
```

### 2. 跨域问题解决方案

通过 Vite 代理配置 解决第三方 API 的跨域限制：
```javascript
// vite.config.ts  
server: {  
  proxy: {  
    '/api': {  
      target: 'https://api.vore.top',  
      changeOrigin: true,  
      rewrite: (path) => path.replace(/^\/api/, '')  
    }  
  }  
}  
```

### 3. 性能优化细节

- 请求防抖：使用 lodash.throttle 限制高频定位请求
- 数据缓存：将非实时的地理位置信息缓存至 localStorage
- 优雅降级：当 API 不可用时，默认显示「神秘访客」并隐藏距离信息

## 三、未来迭代计划

- 天气联动：根据用户城市显示实时天气图标
- 访客统计：生成「今日城市热力图」可视化数据
- 隐私增强：添加 IP 掩码显示选项，强化用户数据保护

## 结语

技术的价值不仅在于功能实现，更在于如何用代码传递人文关怀。这个小小的公告栏就像博客的「玄关」，用技术细节让每个访客感受到「被看见」的温度。

效果：公告栏显示 欢迎来自 河南省郑州市巩义 的小伙伴，告别传统英文地址的疏离感。

### 2. 距离计算：丈量虚拟与现实的尺度

结合 ip-api.com 提供的经纬度数据，运用球面三角公式计算用户与站长的直线距离：

```typescript
function calculateDistance(loc1, loc2) {  
  const R = 6371; // 地球半径（公里）  
  const dLat = (loc2.lat - loc1.lat) * Math.PI / 180;  
  const dLon = (loc2.lng - loc1.lng) * Math.PI / 180;  
  const a = Math.sin(dLat/2) * Math.sin(dLat/2) +  
            Math.cos(loc1.lat*Math.PI/180) * Math.cos(loc2.lat*Math.PI/180) *  
            Math.sin(dLon/2) * Math.sin(dLon/2);  
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));  
}  
```

// 示例：用户与站长同处郑州时，距离显示为 0.00 公里

### 3. 智能问候：用时间编织情感纽带

根据当前小时数自动切换问候语，让每个访问时刻都有专属温度：

```typescript
const hour = new Date().getHours();  
greeting.value = hour < 12 ? '早上好，代码与咖啡缺一不可☕' :  
                  hour < 18 ? '下午好，此刻适合攻克技术难题💻' :  
                  '晚上好，记得抬头看看星空✨';  
```

## 二、技术实现与挑战

### 1. TypeScript 类型安全实践

定义接口规范数据结构，避免运行时字段缺失错误：
```typescript
interface IPLocation {  
  province: string;  
  city: string;  
  district?: string; // 可选区县字段  
  coordinates: { lat: number; lng: number };  
}  
```

### 2. 跨域问题解决方案

通过 Vite 代理配置 解决第三方 API 的跨域限制：

```javascript
// vite.config.ts  
server: {  
  proxy: {  
    '/api': {  
      target: 'https://api.vore.top',  
      changeOrigin: true,  
      rewrite: (path) => path.replace(/^\/api/, '')  
    }  
  }  
}  
```

### 3. 性能优化细节

- 请求防抖：使用 lodash.throttle 限制高频定位请求
- 数据缓存：将非实时的地理位置信息缓存至 localStorage
- 优雅降级：当 API 不可用时，默认显示「神秘访客」并隐藏距离信息

## 三、未来迭代计划

- 天气联动：根据用户城市显示实时天气图标
- 访客统计：生成「今日城市热力图」可视化数据
- 隐私增强：添加 IP 掩码显示选项，强化用户数据保护

## 结语

技术的价值不仅在于功能实现，更在于如何用代码传递人文关怀。这个小小的公告栏就像博客的「玄关」，用技术细节让每个访客感受到「被看见」的温度。