# Tabbar 图标说明

## 图标要求

为了在 App 端和小程序端正常显示 tabbar，需要在此目录下添加以下图标文件：

### 图标文件列表

1. **首页**
   - `home.png` - 未选中状态图标（建议尺寸：81x81px）
   - `home-active.png` - 选中状态图标（建议尺寸：81x81px）

2. **景点**
   - `scenic.png` - 未选中状态图标
   - `scenic-active.png` - 选中状态图标

3. **餐厅**
   - `restaurant.png` - 未选中状态图标
   - `restaurant-active.png` - 选中状态图标

4. **停车场**
   - `parking.png` - 未选中状态图标
   - `parking-active.png` - 选中状态图标

5. **AI助手**
   - `ai.png` - 未选中状态图标
   - `ai-active.png` - 选中状态图标

6. **个人中心**
   - `profile.png` - 未选中状态图标
   - `profile-active.png` - 选中状态图标

## 图标规范

- **格式**：PNG（支持透明背景）
- **尺寸**：建议 81x81px（2倍图）或 162x162px（3倍图）
- **颜色**：未选中状态建议使用灰色，选中状态使用主题色（#3498db）

## 注意事项

- H5 端使用自定义 tabbar 组件，不需要图标文件
- App 端和小程序端需要图标文件才能正常显示 tabbar
- 如果暂时没有图标，可以先用占位图片或纯色图片代替

