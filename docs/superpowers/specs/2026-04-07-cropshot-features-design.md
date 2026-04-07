# CropShot Feature Upgrade Design Spec

## Overview

Nang cap CropShot tu tool cat anh co ban thanh tool cat anh + toi uu export chuyen nghiep. 5 tinh nang moi trong single-file HTML.

## Features

### 1. Ti le crop co dinh (Sidebar)

**Vi tri**: Section moi trong sidebar, duoi gallery, dang chip row.

**Options**: `Free` | `1:1` | `16:9` | `4:3` | `9:16` | `3:2`

**Hanh vi**:
- Default: Free (nhu hien tai)
- Chon ratio -> crop box tu resize giu ti le, anchor center
- Keo corner/edge chi thay doi kich thuoc, giu ratio
- Keo move van tu do
- Chon Free -> tro lai hanh vi hien tai
- State: bien `cropRatio` (null = free, hoac number)

### 2. Chinh anh co ban (Sidebar accordion)

**Vi tri**: Section accordion duoi "Ti le cat" trong sidebar.

**Controls**: 4 slider
- Brightness: 50-150, default 100
- Contrast: 50-150, default 100  
- Saturation: 50-150, default 100
- Sharpen: 0-100, default 0

**Implementation**:
- Brightness/Contrast/Saturation: CSS `filter` tren canvas element (realtime preview)
- Sharpen: convolution kernel, chi apply khi export (performance)
- Nut "Reset" ve gia tri mac dinh
- State: object `adjustments = {brightness:100, contrast:100, saturation:100, sharpen:0}`

### 3. Export Modal

**Trigger**: Nut "Tai anh" tren topbar mo fullscreen modal.

**Layout**: 2 cot
- Trai: Preview anh cat (canvas, max-width/height, checkerboard bg)
- Phai: Panel options

**Options panel**:

a) **Format**: 3 chip `PNG` | `JPG` | `WebP`, default PNG
- PNG: khong co quality slider (lossless)
- JPG/WebP: hien quality slider

b) **Quality**: slider 1-100, default 85, chi hien khi JPG/WebP

c) **Resize**: 
- Toggle bat/tat
- Input width x height (linked ratio lock)
- Preset dropdown: Original, 1920x1080, 1280x720, 800x600, 300x300
- Chon preset -> fill inputs, manual edit -> preset chuyen "Custom"

d) **Dung luong uoc tinh**: 
- Hien thi realtime bang `canvas.toBlob()` callback -> blob.size
- Debounce 300ms khi thay doi quality/resize
- Format: "~245 KB" hoac "~1.2 MB"

e) **Nut "Tai ve"**: lon, gradient tim, o duoi cung

### 4. Paste tu clipboard

**Implementation**:
- `document.addEventListener('paste', handler)`
- Check `e.clipboardData.items` -> tim type bat dau bang 'image/'
- Convert sang blob -> FileReader -> load nhu upload
- Hoat dong o moi trang thai (empty state hoac dang crop)
- Khong can UI them

### 5. Paste - UX

- Khi paste thanh cong: load anh vao workspace nhu upload
- Khi paste khong co anh: khong lam gi (silent fail)

## UI/UX Notes

- Sidebar sections dung accordion voi chevron icon (Lucide SVG)
- Accordion default: "Ti le cat" mo, "Chinh anh" dong
- Export modal dong bang nut X, click backdrop, hoac Esc
- Tat ca slider dung style consistent voi zoom slider hien tai
- Mau sac: thuan xanh tim den, khong xanh la

## Technical Notes

- Tat ca trong 1 file index.html
- CSS filter cho realtime preview (khong re-render canvas moi frame)
- Sharpen chi khi export (canvas convolution expensive)
- toBlob() cho estimate file size (async, debounce)
- Giu nguyen drag system hien tai, chi them ratio constraint logic

## Out of scope

- Batch processing
- Undo/redo
- Watermark
- History/localStorage
