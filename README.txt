排版编辑器 · 源码包
====================

一、目录内容
------------
editor-source.html     全部代码（HTML + CSS + JS 都在里面，约 3900 行）
fonts.css      字体声明
fonts\         字体文件（站酷快乐体、站酷黄油体、Aharoni 等）
README.txt     本文件

二、怎么运行
------------
双击 editor-source.html 即可（数据会存在你当前使用的浏览器里）。
注意：数据是按「浏览器」存的，换浏览器 / 清浏览数据会看不到之前的项目。

三、代码导览（面试讲这个就够）
------------------------------
1. 整体结构
   - 单个 <style>：所有样式
   - 单个 <script>：所有逻辑，没有模块、没有构建步骤
   - 数据模型：state = { pages:[{id,name,w,h,music,items:[...]}], assets:[...], currentPageId }

2. 渲染
   - renderStage()：按当前页的 items 数组**全量重建 DOM**
   - applyItem(el, it)：left = x - w/2、top = y - h/2、width/height、transform: rotate()
   - 元素的 x/y 存的是**中心点坐标**

3. 交互手势
   - stage 上监听 pointerdown，用 setPointerCapture 接管后续 move/up
   - pointToStage()：屏幕坐标 ÷ 画布缩放比 scale → 画布坐标
   - 缩放（resize）和旋转（rotate）共用同一套指针逻辑，靠 data-handle 区分

4. 存储
   - localStorage['layout-editor.v1']：项目结构（JSON，几十 KB）
   - IndexedDB 'layout-editor-assets' / store 'blobs'：图片、音频二进制
   - 渲染时 URL.createObjectURL(blob) 得到 blob: 地址
   - saveNow()：260ms 防抖写盘；另有最多 5 份历史快照（.h1~.h5）

5. 撤销 / 重做
   - pushHistory()：操作前把 serialize 后的状态压进 history 数组
   - undo/redo：弹栈 → normalize → renderAll

6. 多窗口同步
   - BroadcastChannel('layout-editor') + window 'storage' 事件
   - 用 updatedAt 时间戳判断谁更新，整份覆盖（**已知缺陷：后写覆盖**）

7. 交互系统（元素的 it.act）
   - type: page / back / url / toggle / cycle / scroll
   - 预览时生效；cycle 用 _pvHidden / _pvShown 等临时标记，退出预览时删除，不写进存档
   - 自动播放（轮播）用 setInterval；自动消失用 setTimeout

8. 动画
   - 进场动画：CSS @keyframes + animation-delay/duration
   - 持续动画（旋转/浮动）：早期用 CSS keyframes，网页版改成 requestAnimationFrame 逐帧计算
   - 悬浮放大：CSS :hover + !important（因为 CSS 动画会压过普通声明）

9. 导出网站
   - 把素材转 dataURL 内联，拼一个自包含 HTML
   - 图片用 OffscreenCanvas 压成 WebP（55MB → 4.6MB）

四、已知不足
----------------------------------
1. 全量重建 DOM，元素多时性能一般 —— 可改增量更新
2. 多窗口同步是"最后写入者赢"，应改为按 item 合并 / 版本号
3. 单文件导出体积大（音视频 base64 会膨胀 1/3）
4. 没有做移动端手势（双指缩放/旋转）
5. 撤销是整份快照，数据大时占内存

五、本项目使用 AI 辅助开发
--------------------------
需求、界面效果、测试和 bug 定位由本人负责，代码主要由 AI 生成。
本人能解释上面列出的核心逻辑，也清楚第四节的不足。
