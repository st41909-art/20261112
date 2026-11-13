# 真實風格 FPS 視覺規格書

## 1. 技術棧
- Three.js r170+，WebGPURenderer 優先，fallback 到 WebGL2
- 單一 index.html，所有 shader 內嵌，CDN 載入 Three.js
- 目標裝置：iPhone 15 Pro 以上、Snapdragon 8 Gen 3 以上，60fps 穩在 1080p
- 場景規模：可見距離 2 公里，LOD 四層
- 貼圖格式：KTX2 壓縮，Basis Universal
- 音效：Web Audio API，3D 空間音訊

## 2. 風格定義
- 2026 年真實世界能存在的東西
- 明確排除：科幻、賽博龐克、奇幻、末日廢土
- 參考：《三角洲行動》的寫實都市與郊區畫面、《Escape from Tarkov》的工業與建築細節
- 光線：自然日光為主，硬陰影、長投影、大氣散射
- 色調：去飽和但保留真實材質的局部高光，避免 HDR 過曝感

## 3. 場景（四種可切換，URL 參數切換）

### 3.1 都市
- 一條 400 公尺街道，兩側 8-12 棟建築
- 建築：現代水泥、玻璃幕牆、紅磚老宅混合
- 細節：破損窗戶、垂落電線、消防栓、路燈、垃圾桶、路邊停車
- 地面：瀝青、人行道磚、積水坑（有漣漪 shader）
- 天空：程序化雲層 + 真實方位的太陽位置

### 3.2 郊區
- 住宅區，獨棟或雙拼房屋 6-10 棟
- 細節：車庫、后院圍籬、晾衣繩、郵筒、雜草叢生的路邊
- 地面：柏油、碎石 driveway

### 3.3 農村
- 農田、農舍、穀倉、圍欄
- 細節：乾草堆、農機具、泥土路、水溝
- 地面：泥土、草地（grass cards 或 GPU grass）

### 3.4 工業區
- 工廠建築、貨櫃、起重機、軌道
- 細節：鏽蝕鐵皮、管道、警示色、廢棄棧板
- 地面：水泥、油污痕跡

## 4. 角色
- 玩家：第一人稱，只有手和武器可見
- 敵人：6-10 個 AI 士兵，分 2-3 個陣營
- 模型：Quaternius 或 Kenney 軍用低模為基礎，重寫材質
  - 皮膚：次表面散射（SSS），毛細孔 normal map
  - 服裝：織物 normal + 髒污遮罩 + 各向異性高光（濕布料）
  - 裝備：戰術背心、頭盔、背包，PBR 多通道
- 動作：Mixamo 循環，60fps 以上
  - 待機、跑步、掩體切換（左右 Peek）、開槍、換彈、死亡
  - 混合樹處理過渡，避免腳滑

## 5. 武器
- 3-5 把現代槍械：突击步枪、精確射手步枪、手槍、霰彈槍
- 模型：CC0 現代槍械包（如 Poly Pizza、Quaternius）
- 材質分層：金屬（拉絲/鏡面）、聚合物、橡膠、木質握把
- 動畫：開槍後座、槍口火焰（billboard + 粒子）、煙霧拖尾
- 彈道：hitscan 加視覺彈道線（tracer），命中有彈孔 decal 累積

## 6. 燈光
- 主方向光（太陽）：色溫 5500K，硬陰影，cascade shadow map 三層
- 天空光：hemispheric + 程序化天空 dome
- 點光：路燈、車燈、建築窗戶漏光，數量上限 16
- 體積光：陽光穿過灰塵的 god rays（radial blur 或 screen-space）
- 環境光遮蔽：SSAO 或 HBAO

## 7. 後製
- 色調映射：ACES Filmic
- 泛光：bloom，只對高亮（燈、火焰、金屬反射）
- 景深：auto-focus 到準星附近
- 動態模糊：攝影機移動和後座力時
- 暗角：vignette，開火時加強
- 顆粒：film grain，低強度
- 色差：chromatic aberration，極低

## 8. 性能預算
- Draw calls：< 150 per frame
- Triangles：< 500K visible
- 貼圖記憶體：< 512MB
- 粒子上限：300
- 物件實例化：建築、植被、武器零件
- 視錐剔除：frustum culling + occlusion culling
- 陰影：cascade，只對玩家附近 50 公尺高解析度

## 9. 禁止事項
- 禁止 AI 生成 3D 模型
- 禁止手寫超高面數模型
- 禁止非 PBR 的假材質（純 color + emissive 湊合）
- 禁止缺少 normal/roughness/metalness 的材質
- 禁止 2D sprite 代替 3D 角色
- 禁止明顯的低模感（flat shading、無細節的大平面）

## 10. 交付格式
- 單一 index.html，可直接在瀏覽器打開
- 所有 shader 以 <script type="x-shader/x-fragment"> 和 x-vertex 內嵌
- 所有貼圖以 base64 或 KTX2 內嵌，或從 CDN 載入
- 提供 README 說明如何切換場景（URL 參數）
- 提供效能測試模式（顯示 FPS、draw calls、triangle count）