Android记事本应用开发实验报告
一、实验概述
1.1 实验目的
本次实验旨在开发一个功能完整的Android记事本应用，通过实践掌握Android应用开发的核心技术，包括界面设计、数据存储、用户交互等方面。应用要求实现笔记的增删改查、模糊搜索和时间戳显示等基本功能。

1.2 实验环境
开发工具：Android Studio Hedgehog | 2023.1.1
开发语言：Java，Kotlin
目标平台：Android API 21及以上
数据库：SQLite

二、项目结构设计
2.1 项目架构
该项目采用MVVM（Model-View-ViewModel）架构结合模块化设计：

data层：NotepadRepository和PreferenceManager负责数据管理
viewmodel层：NotepadViewModel处理UI相关数据
ui层：包含components、content、routes等，采用声明式UI
di层：依赖注入管理，提高代码可测试性
基于styles_notepad.xml和colors.xml，应用采用Material Design 3设计规范，提供现代化的视觉体验。

三、核心功能实现
3.1 数据模型设计
创建Note类作为数据实体，包含id、标题、内容和时间戳四个核心属性。采用标准的Java Bean设计模式，为每个属性提供getter和setter方法，确保数据的封装性和可访问性。

3.2 数据库管理
使用SQLiteOpenHelper创建和管理本地数据库。设计notes表包含_id、title、content和timestamp四个字段。实现了完整的CRUD操作：

create：通过insert方法添加新笔记
read：通过query方法查询笔记列表和搜索功能
update：通过update方法修改现有笔记
delete：通过delete方法删除指定笔记

四、核心界面组件设计
2.1 主界面（NotepadActivity）
主界面布局结构：
├── 顶部应用栏 (AppBar)
│   ├── 应用标题
│   ├── 搜索按钮
│   ├── 设置菜单
│   └── 新建笔记FAB
├── 笔记列表 (RecyclerView/LazyColumn)
│   └── 笔记卡片 (Card)
│       ├── 笔记标题
│       ├── 内容预览（截断显示）
│       ├── 时间戳
│       └── 标签/分类标识
└── 底部导航栏 (可选)

2.2 独立编辑界面（StandaloneEditorActivity）
编辑界面布局：
├── 工具栏 (Rich Toolbar)
│   ├── 返回/保存按钮
│   ├── 格式设置（粗体、斜体、列表）
│   ├── 附件添加
│   └── 更多选项
├── 标题输入区 (EditText)
├── 内容编辑区 (Rich Editor)
│   └── 支持富文本编辑
└── 底部状态栏
    └── 字数统计/自动保存状态
    
三、导航与路由系统
3.1 路由结构（基于ui/routes包）
// 预期的导航路由
sealed class NavRoute(val route: String) {
    object NoteList : NavRoute("note_list")
    object NoteDetail : NavRoute("note_detail/{noteId}")
    object NoteEdit : NavRoute("note_edit/{noteId}")
    object Settings : NavRoute("settings")
    object Export : NavRoute("export")
}

3.2 导航模式
主从布局：在平板上可能采用主列表+详情并排显示
堆栈导航：在手机上采用标准的堆栈导航模式
深链接支持：通过NavState.kt管理复杂的导航状态

四、特色界面功能
4.1 快捷操作（基于shortcuts.xml）
应用快捷方式：
├── 新建笔记
├── 搜索笔记
├── 最近编辑
└── 特定笔记直达

4.2 主题系统（基于SystemTheme.kt）
// 支持的主题模式
enum class ThemeMode {
    SYSTEM_DEFAULT,
    LIGHT,
    DARK,
    AUTO_NIGHT
}

4.3 键盘快捷键（KeyboardShortcuts.kt）
针对外接键盘设备优化：
Ctrl+N：新建笔记
Ctrl+F：搜索
Ctrl+S：保存
Ctrl+Z：撤销

五、组件化界面设计
5.1 可复用组件（ui/components包）
组件库：
├── NoteCardComponent - 笔记卡片
├── SearchBarComponent - 搜索栏
├── RichTextEditor - 富文本编辑器
├── TagChipGroup - 标签选择
├── DatePicker - 时间选择
└── ConfirmationDialog - 确认对话框

5.2 内容预览（ui/content包）
// 预览组件用于开发时快速迭代
@Preview
@Composable
fun NoteCardPreview() {
    NoteCardComponent(note = sampleNote)
}

六、数据驱动的界面设计
6.1 响应式布局
基于NotepadViewModel.kt的状态管理：
data class NotepadUiState(
    val notes: List<Note> = emptyList(),
    val searchQuery: String = "",
    val selectedNote: Note? = null,
    val isLoading: Boolean = false,
    val error: String? = null
)

6.2 偏好设置界面
通过PreferenceManager管理：
字体大小设置
自动保存间隔
默认笔记格式
导出选项设置

七、现代化设计特征
7.1 动态颜色（Material You）
基于colors.xml推断支持：
动态取色系统
个性化主题
高对比度模式支持

7.2 无障碍设计
通过ContextExtensions.kt提供：
屏幕阅读器支持
大字体兼容
高对比度文本

7.3 交互动画
基于usecase包的功能：
平滑的页面转场
列表项动画
加载状态指示器

八、多场景适配
8.1 多窗口支持
通过NotepadActivity配置：
分屏模式适配
画中画模式（可能）
自由窗体支持

8.2 导出功能界面
基于ArtVandelay.kt（导出相关）：
导出选项：
├── 格式选择（TXT、PDF、HTML）
├── 导出范围（全部、选中）
├── 目标位置选择
└── 进度显示界面

五、实验过程与问题解决
5.1 开发环境配置阶段

问题1：Gradle同步失败，出现依赖冲突
分析：新版本Android Studio使用Version Catalog管理依赖，部分库版本要求较高的编译SDK
解决方案：
将compileSdk从34降级到33以保持兼容性
明确指定依赖库版本，避免自动解析带来的冲突
对于Navigation等非必需组件，选择移除以简化项目结构

问题2：自适应图标兼容性问题
分析：mipmap-anydpi-v26中的自适应图标需要API 26以上支持
解决方案：
删除mipmap-anydpi和mipmap-anydpi-v26文件夹
使用传统的PNG图标资源
保持minSdk为21以支持更多设备

5.2 核心功能实现阶段
问题3：数据库版本管理
分析：应用升级时数据库结构可能发生变化
解决方案：
在onUpgrade方法中处理数据库版本迁移
使用常量管理数据库版本号
采用稳健的数据备份策略

问题4：列表性能优化
分析：大量笔记数据可能导致列表滚动卡顿
解决方案：
实现ViewHolder模式复用视图
使用DiffUtil优化数据更新
对长内容进行截断显示

5.3 用户体验优化阶段
问题5：时间显示格式
分析：默认时间格式不符合中文用户习惯
解决方案：
自定义SimpleDateFormat格式
使用Locale.CHINA确保本地化显示
在编辑时自动更新时间戳

问题6：搜索功能实现
分析：需要实现实时搜索和高亮显示
解决方案：
使用SearchView组件
实现TextWatcher监听输入变化
优化数据库查询性能

六、实验成果与总结
6.1 功能实现情况
成功实现了所有预定功能：
✓ 笔记的增删改查操作
✓ 模糊搜索功能
✓ 时间戳显示
✓ 用户友好的界面交互
✓ 数据持久化存储

6.2 技术收获
通过本次实验，深入掌握了：
Android应用架构设计方法
SQLite数据库操作与管理
RecyclerView的高级用法
材料设计规范的实践应用
问题排查和调试技巧
