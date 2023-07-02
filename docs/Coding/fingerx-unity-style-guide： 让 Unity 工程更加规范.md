> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [github.com](https://github.com/fingerx/unity-style-guide)

> 让 Unity 工程更加规范. Contribute to fingerx/unity-style-guide development by creating an account on GitHub.

[](#unity工程规范-)Unity 工程规范 () {
==============================

_最合理的 Unity 规范_

灵感来源于 Airbnb 的 JS 规范 [Airbnb Javascript Style Guide](https://github.com/airbnb/javascript).

[](#专业术语)专业术语
-------------

##### [](#levelsmapscene关卡地图场景)Levels/Map/Scene(关卡 / 地图 / 场景)

“map”(地图)这个词通常也会被称为 “level”(关卡) 或者“Scene”(场景)，三者的含义是等同的，

在 Unity 惯用 Scenes 文件夹放置所有的 Level 文件。

##### [](#cases大小写)Cases(大小写）

对于字母大小写的规范有数种，以下是几种常见的几种

> ###### [](#pascalcase)PascalCase
> 
> 每个单词的首字母都是大写，单词之间没有其他字符，例如 ：`DesertEagle`, `StyleGuide`, `ASeriesOfWords`。
> 
> ###### [](#camelcase)camelCase
> 
> 第一个单词的首字母小写，后面的单词的首字母大写，例如：`desertEagle`, `styleGuide`, `aSeriesOfWords`。
> 
> ###### [](#snake_case)Snake_case
> 
> 单词之间用下划线链接，单词的首字母可以大写也可以小写，例如：`desert_Eagle`, `Style_Guide`, `a_Series_of_Words`

##### [](#variables--properties--field变量属性字段)Variables / Properties / Field(变量 / 属性 / 字段)

'变量'和'属性'和'字段'三个词在很多情况下是可以互相通用的。但如果他们同时出现在一个环境时，含义有一些不同：

###### [](#fields-字段)fields (字段)

'字段'通常定义在一个类的内部。例如，如果一个类`PlayerFollower`有一个内部成员`bFollowing`，那么`bFollowing`可以是做是`PlayerFollower`的一个字段

###### [](#property-属性)Property (属性)

'属性'相当于是给字段加了一个保护套，如果想读这个字段的值，属性里面走的一定是 get{}，如果想给字段赋值，属性里一定走的是 set{}，那么程序员可以在 get{} 和 set{} 中增加一些限制，验证要赋值的内容。

当'属性'用在类的内部时，通常用来获取已经定义好的数据，比如 PlayerController.Pawn(属性) 拿到正在控制对象。

###### [](#variable-变量)Variable (变量)

'变量'通常用在给函数传递参数，或者用在函数内的局部变量

当'变量'用在类的内部时，通常是用来定义什么或者用来保存某些数据的 (比如对组件的各种引用，或者一些可编辑项)。

[](#0-原则)0. 原则
--------------

以下原则改编自 [idomatic.js 代码规范](https://github.com/rwaldron/idiomatic.js/)。

### [](#01-如果你的项目已经存在现有规范那么请继续遵守这套规范)0.1 如果你的项目已经存在现有规范，那么请继续遵守这套规范。

如果你工作的项目或你的团队已经有一套自己的规范，那么请尊重它。如果现有规范和本套规范发生冲突，请继续遵守原规范。

好的项目规范应该是不断进步的，当你发现有好的更改可以适合所有用户时，你应该建议去更改现有规范

> #### [](#对规范优劣的争论是没有意义的有规范你就该去遵守)"对规范优劣的争论是没有意义的，有规范你就该去遵守。"
> 
> [_Rebecca Murphey_](https://rmurphey.com)

### [](#02-不管团队中有多少人工程中所有的数据结构资源代码风格应该统一如同是同一个人的作品)0.2 不管团队中有多少人，工程中所有的数据结构、资源、代码风格应该统一，如同是同一个人的作品

把资源从一个工程迁移到另一个工程不应该产生新的学习成本，所有资源应该符合项目规范，消除不必要的歧义和不确定性

遵守规范可以促进对于项目的生产和维护效率，因为团队成员不用去考虑规范问题，只需要去遵守。本套规范根据诸多优秀经验编写，遵守它意味着可以少走很多弯路。

### [](#03-好哥们也得讲原则)0.3 好哥们也得讲原则。

如果你发现同事不遵守规范，你该去纠正他

当你在团队中工作时，或者在社区提问时 (例如 [Unreal Slackers](http://join.unrealslackers.org/))，你会发现如果大家都遵守同一套规范会容易的多，没有人愿意在面对一堆乱糟糟的蓝图或者莫名其妙的代码时帮你解决问题。

如果你要帮助的人使用另一套不同但很健全的规范，你应该去适应它，如果他们没有遵守任何规范，那么带他们来这儿。

### [](#04-没有规范的团队不是真正的团队)0.4 没有规范的团队不是真正的团队

当你要加入一个新的 UE4 团队时，你应该首先问他们有没有项目规范，如果没有的话，你该怀疑他们是否有能力像一个真正的团队那样工作

[](#目录)目录
---------

> 1.  [资源命名约定](#anc)
> 2.  [目录结构](#structure)
> 3.  [代码](#bp)

[](#1-资源命名约定-)1. 资源命名约定 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

对于资源的命名的规范应该像法律一样被遵守。一个项目如果有好的命名规范，那么在资源管理、查找、解析、维护时，都会有极大的便利性。

大多数资源的命名都应该有前缀，前缀一般是资源类型的缩写，然后使用下划线和资源名链接。

### [](#11-基本命名规则-prefix_baseassetname_variant_suffix-)1.1 基本命名规则 `Prefix_BaseAssetName_Variant_Suffix` [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

时刻记住这个命名规范`Prefix_BaseAssetName_Variant_Suffix`，只要遵守它，大部分情况下都可以让命名规范。下面是详细的解释。

`Prefix`(前缀) 和 `Suffix`(后缀) 由资源类型确定，请参照下面的[资源类型表](#asset-name-modifiers)。

所有资源都应该有一个`BaseAssetName`(基本资源名)。所谓基本资源名表明该资源在逻辑关系上属于那种资源，任何属于该逻辑组的资源都应该遵守同样的命名规范

基本资源名应该使用简短而便于识别的词汇，例如，如果你有一个角色名字叫做 Bob，那么所有和 Bob 相关的资源的`BaseAssetName`都应该叫做`Bob`

`Varient`(扩展名) 用来保证资源的唯一性，同样，扩展名也应该是简短而容易理解的短词，以说明该资源在所属的资源逻辑组中的子集。例如，如果 Bob 有多套皮肤，那么这些皮肤资源都应该使用 Bob 作为基本资源名同时包含扩展名，例如'Evil'类型的皮肤资源，名字应该是`Bob_Evil`，而 Retro 类型的皮肤应该是用`Bob_Retro`

一般来说，如果仅仅是为了保证资源的唯一性，`Varient`可以使用从`01`开始的两位数字来表示。例如，如果你要制作一堆环境中使用的石头资源，那么他们应该命名为`Rock_01`, `Rock_02`, `Rock_03`等等。除非特殊需要，不要让数字超过三位数，如果你真的需要超过 100 个的资源序列，那么你应该考虑使用多个基础资源名

基于你所制作的资源扩展属性，你可以把多个扩展名串联起来。例如，如果你在制作一套地板所使用的资源，那么你的资源除了使用`Flooring`作为基本名，扩展名可以使用多个，例如`Flooring_Marble_01`, `Flooring_Maple_01`, `Flooring_Tile_Squares_01`。

#### [](#11-范例)1.1 范例

##### [](#11e1-bob)1.1e1 Bob

<table><thead><tr><th>资源类型</th><th>资源名</th></tr></thead><tbody><tr><td>Skinned Mesh</td><td>SK_Bob</td></tr><tr><td>Material</td><td>M_Bob</td></tr><tr><td>Texture (Diffuse/Albedo)</td><td>T_Bob_D</td></tr><tr><td>Texture (Normal)</td><td>T_Bob_N</td></tr><tr><td>Texture (Evil Diffuse)</td><td>T_Bob_Evil_D</td></tr></tbody></table>

##### [](#11e2-rocks石头)1.1e2 Rocks（石头）

<table><thead><tr><th>资源类型</th><th>资源名</th></tr></thead><tbody><tr><td>Static Mesh (01)</td><td>S_Rock_01</td></tr><tr><td>Static Mesh (02)</td><td>S_Rock_02</td></tr><tr><td>Static Mesh (03)</td><td>S_Rock_03</td></tr></tbody></table>

### [](#12-资源类型表-)1.2 资源类型表 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

当给一个资源命名的时候，请参照以下表格来决定在[基本资源名](#base-asset-name)前后所使用的前缀和后缀

#### [](#目录-1)目录

> 1.2.1 通用类型 [Most Common](#anc-common)

> 1.2.2 动作 [Animations](#anc-animations)

> 1.2.5 材质 [Materials](#anc-materials)

> 1.2.6 纹理 [Textures](#anc-textures)

> 1.2.7 杂项 [Miscellaneous](#anc-misc)

> 1.2.8 精灵 [Sprites](#anc-paper2d)

> 1.2.9 物理 [Physics](#anc-physics)

> 1.2.10 声音 [Sound](#anc-sound)

> 1.2.11 界面 [User Interface](#anc-ui)

> 1.2.12 特效 [Effects](#anc-effects)

#### [](#121-通用类型-)1.2.1 通用类型 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Level / Map</td><td></td><td></td><td><a href="#2.4">所有地图应该放在 Scenes/Maps 目录下</a></td></tr><tr><td>Level (Persistent)</td><td></td><td>_P</td><td>_P 表示是一个大的容器 Level，在这个 Level 里可以异步加载其他小 Level</td></tr><tr><td>Level (Audio)</td><td></td><td>_Audio</td><td></td></tr><tr><td>Level (Lighting)</td><td></td><td>_Lighting</td><td></td></tr><tr><td>Level (Geometry)</td><td></td><td>_Geo</td><td></td></tr><tr><td>Level (Gameplay)</td><td></td><td>_Gameplay</td><td></td></tr><tr><td>Material</td><td>M_</td><td></td><td></td></tr><tr><td>Static Mesh</td><td>S_ or SM_</td><td></td><td>选一个，建议使用 S_.</td></tr><tr><td>Skinned Mesh</td><td>SK_</td><td></td><td></td></tr><tr><td>Texture</td><td>T_</td><td>_?</td><td>参照<a href="#anc-textures">纹理</a></td></tr><tr><td>Particle System</td><td>PS_</td><td></td><td></td></tr><tr><td>Canvas</td><td>UI_</td><td></td><td></td></tr></tbody></table>

#### [](#122-动作-)1.2.2 动作 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Animator Controller</td><td>AC_</td><td></td><td></td></tr><tr><td>Animator Override Controller</td><td>AOC_</td><td></td><td></td></tr><tr><td>Animation</td><td>AM_</td><td></td><td></td></tr><tr><td>Skinned Mesh</td><td>SK_</td><td></td><td></td></tr></tbody></table>

### [](#125-材质-)1.2.5 材质 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Material</td><td>M_</td><td></td><td></td></tr><tr><td>Shader</td><td>S_/SD_</td><td></td><td>建议 SD_</td></tr><tr><td>Physical Materials</td><td>PM_</td><td></td><td></td></tr></tbody></table>

### [](#126-纹理-)1.2.6 纹理 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Texture</td><td>T_</td><td></td><td></td></tr><tr><td>Texture (Diffuse/Albedo/Base Color)</td><td>T_</td><td>_D</td><td></td></tr><tr><td>Texture (Normal)</td><td>T_</td><td>_N</td><td></td></tr><tr><td>Texture (Roughness)</td><td>T_</td><td>_R</td><td></td></tr><tr><td>Texture (Alpha/Opacity)</td><td>T_</td><td>_A</td><td></td></tr><tr><td>Texture (Ambient Occlusion)</td><td>T_</td><td>_O or _AO</td><td>选一个，建议使用 _O.</td></tr><tr><td>Texture (Bump)</td><td>T_</td><td>_B</td><td></td></tr><tr><td>Texture (Emissive)</td><td>T_</td><td>_E</td><td></td></tr><tr><td>Texture (Mask)</td><td>T_</td><td>_M</td><td></td></tr><tr><td>Texture (Specular)</td><td>T_</td><td>_S</td><td></td></tr><tr><td>Texture (Packed)</td><td>T_</td><td>_*</td><td></td></tr><tr><td>Texture Cube</td><td>TC_</td><td></td><td></td></tr><tr><td>Media Texture</td><td>MT_</td><td></td><td></td></tr><tr><td>Render Texture</td><td>RT_</td><td></td><td></td></tr><tr><td>Cube Render Texture</td><td>RTC_</td><td></td><td></td></tr></tbody></table>

#### [](#1261-纹理合并-)1.2.6.1 纹理合并 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

把多张纹理存于一个纹理文件中是很常见的方法，比如通常可以把自发光 (Emissive), 粗糙度(Roughness), 环境光(Ambient Occlusion) 以 RGB 通道的形式保存在纹理中，然后在文件的后缀中，注明这些信息，例如`_EGO`

> 一般来说，在纹理的 Diffuse 信息中附带 Alpha/Opacity 信息是很常见的，这时在`_D`后缀中可以加入`A`也可以不用加

不推荐同时把 RGBA 四个通道的信息保存在一张纹理中，这是由于带有 Alpha 通道的纹理要比不带的占用更多的资源，除非 Alpha 信息是以蒙版 (Mask) 的形式保存在 Diffuse/Albedo 通道中。

### [](#127-杂项-)1.2.7 杂项 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Playables Behavior</td><td>PB_</td><td></td><td></td></tr><tr><td>Playables Assets</td><td>PA_</td><td></td><td></td></tr></tbody></table>

### [](#128-2d-)1.2.8 2D [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Sprite</td><td>SPR_/SP_</td><td></td><td></td></tr><tr><td>Sprite Atlas Group</td><td>SPRG_/SPG_</td><td></td><td></td></tr><tr><td>Tile Map</td><td>TM_</td><td></td><td></td></tr></tbody></table>

### [](#129-物理-)1.2.9 物理 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Physical Material</td><td>PM_</td><td></td><td></td></tr><tr><td>Destructible Mesh</td><td>DM_</td><td></td><td></td></tr></tbody></table>

### [](#1210-声音-)1.2.10 声音 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Audio Mixer</td><td>Mix_</td><td></td><td></td></tr><tr><td>Sound Wave</td><td>A_</td><td></td><td></td></tr></tbody></table>

### [](#1211-界面-)1.2.11 界面 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>资源类型</th><th>前缀</th><th>后缀</th><th>备注</th></tr></thead><tbody><tr><td>Font</td><td>Font_</td><td></td><td></td></tr></tbody></table>

### [](#1212-effects-)1.2.12 Effects [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

<table><thead><tr><th>Asset Type</th><th>Prefix</th><th>Suffix</th><th>Notes</th></tr></thead><tbody><tr><td>Particle System</td><td>PS_</td><td></td><td></td></tr><tr><td>Material (Post Process)</td><td>PP_</td><td></td><td></td></tr></tbody></table>

[](#2-目录结构-)2. 目录结构 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

对资源目录的规范管理和资源文件同等重要，都应该像法律一样被严格遵守。不规范的目录结构会导致严重的混乱。

有多种不同管理 Unity 资源目录的方法，在本套规范中，我们尽量利用了 Unity 的 Project 视图的过滤和搜索功能来查找资源，而不是按照资源类型来划分目录结构。

> 如果你正确遵守了前面使用前缀的资源[命名规范](#1.2)，那么就没有必要按照资源类型创建类似于`Meshes`, `Textures`, 和 `Materials`这样的目录结构，因为你可以在过滤器中通过前缀来找到特定类型的资源

### [](#2e1-目录结构示例)2e1 目录结构示例

```
|-- Assets
    |-- Fairytale
        |-- Art
        |   |-- Industrial
        |   |   |-- Ambient
        |   |   |-- Machinery
        |   |   |-- Pipes
        |   |-- Nature
        |   |   |-- Ambient
        |   |   |-- Foliage
        |   |   |-- Rocks
        |   |   |-- Trees
        |   |-- Office
        |-- Characters
        |   |-- Bob
        |   |-- Common
        |   |   |-- Animations
        |   |   |-- Audio
        |   |-- Jack
        |   |-- Steve
        |   |-- Zoe
        |-- Core
        |   |-- Characters
        |   |-- Engine
        |   |-- GameModes
        |   |-- Interactables
        |   |-- Pickups
        |   |-- Weapons
        |-- Effects
        |   |-- Electrical
        |   |-- Fire
        |   |-- Weather
        |-- Scenes
        |   |-- Level1
        |   |-- Level2
        |-- MaterialLibrary
        |   |-- Debug
        |   |-- Metal
        |   |-- Paint
        |   |-- Utility
        |   |-- Weathering
        |-- Placeables
        |   |-- Pickups
        |-- Weapons
            |-- Common
            |-- Pistols
            |   |-- DesertEagle
            |   |-- RocketPistol
            |-- Rifles


```

使用这种目录结构的原因列在下面

### [](#目录-2)目录

> 2.1 文件夹命名 [Folder Names](#structure-folder-names)

> 2.2 顶层目录 [Top-Level Folders](#structure-top-level)

> 2.3 开发者目录 [Developer Folders](#structure-developers)

> 2.4 地图目录 [Scenes](#structure-Scenes)

> 2.5 核心资源 [Core](#structure-core)

> 2.6 [避免以`Assets` 或 `AssetTypes`命名](#structure-assettypes)

> 2.7 超大资源 [Large Sets](#structure-large-sets)

> 2.8 材质库 [Material Library](#structure-material-library)

### [](#21-文件夹命名-)2.1 文件夹命名 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

关于文件夹的命名，有一些通用的规范

#### [](#211-使用pascalcase大小写规范-)2.1.1 使用 PascalCase 大小写规范 [*](#terms-cases) [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

文件夹的命名需要遵守 PascalCase 规范，也就是所有单词的首字母大写，并且中间没有任何连接符。例如`DesertEagle`, `RocketPistol`, and `ASeriesOfWords`.

参照[大小写规范](#terms-cases).

#### [](#212-不要使用空格-)2.1.2 不要使用空格 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

作为对 [2.1.1](#2.1.1) 的补充，绝对不要在目录名中使用空格。空格会导致引擎以及其他命令行工具出现错误，同样，也不要把你的工程放在包含有空格的目录下面，应该放在类似于`D:\Project`这样的目录里，而不是`C:\Users\My Name\My Documents\Unreal Projects`这样的目录。

#### [](#213-不要使用其他unicode语言字符或奇怪的符号-)2.1.3 不要使用其他 Unicode 语言字符或奇怪的符号 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

如果你游戏中的角色的名字叫做'Zoë'，那么文件夹要其名为`Zoe`。在目录名中使用这样的字符的后果甚至比使用空格还严重，因为某些引擎工具在设计时压根就没有考虑这种情况。

顺便说一句，如果你的工程碰到了类似于[这篇帖子](https://answers.unrealengine.com/questions/101207/undefined.html)中的情况，并且当前使用的系统用户名中包含有 Unicode 字符（比如 `Zoë`），那么只要把工程从`My Documents`目录移到类似于`D:\Project`这种简单的目录里就可以解决了。

记住永远在目录名中只使用`a-z`, `A-Z`, 和 `0-9`这些安全的字符，如果你使用了类似于`@`, `-`, `_`, `,`, `*`, 或者 `#`这样的字符，难免会碰到一些操作系统、源码管理工具或者一些弱智的工具让你吃个大亏。

### [](#22-使用一个顶级目录来保存所有工程资源-)2.2 使用一个顶级目录来保存所有工程资源 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

所有的工程资源都应该保存在一个以工程名命名的目录中。例如你有一个工程叫做'Fairytale'，那么所有该工程的资源都应该保存在`Assets/Fairytale`目录中。

> 开发者目录`Developers`不用受此限制，因为开发者资源是以实验目的使用的，参照下面的[开发者目录](#2.3)中的详细说明。

使用顶级目录的原因有很多。

#### [](#221-避免全局资源)2.2.1 避免全局资源

通常在代码规范中会警告你不要使用全局变量以避免污染全局命名空间。基于同样的道理，不存在于工程目录中的资源对于资源管理会造成不必要的干扰。

每个属于项目资源都应该有它存在的目的。如果仅仅是为了测试或者体验而使用的资源，那么这些资源应该放在[`开发者`](#2.3)目录中。

#### [](#222-减少资源迁移时的冲突)2.2.2 减少资源迁移时的冲突

当一个团队有多个项目时，从一个项目中把资源拷贝到另一个项目会非常频繁，这时最方便的方法就是使用 Unity 的包导入导出功能，因为这个功能会把资源的依赖项一起拷贝到目标项目中。

这些依赖项经常造成麻烦。如果两个工程没有项目顶级目录，那么这些依赖项很容易就会被拷贝过来的同名资源覆盖掉，从而造成意外的更改。

##### [](#222e1-举例基础材质的麻烦)2.2.2e1 举例：基础材质的麻烦

举个例子，你在一个工程中创建了一个基础材质，然后你把这个材质迁移到了另一个工程中。如果你的资源结构中没有顶级目录的设计，那么这个基础材质可能放在`Assets/MaterialLibrary/M_Master`这样的目录中，如果目标工程原本没有这个材质，那么很幸运暂时不会有麻烦。

随着两个工程的推进，有可能这个基础材质因工程的需求不同而发生了不同的修改。

问题出现在，其中一个项目的美术制作了一个非常不错的模型资源，另一个项目的美术想拿过来用。而这个资源使用了`Assets/MaterialLibrary/M_Master`这个材质，那么当迁移这个模型时，`Assets/MaterialLibrary/M_Master`这个资源就会出现冲突。

这种冲突难以解决也难以预测，迁移资源的人可能压根就不熟悉工程所依赖的材质是同一个人开发的，也不清楚所依赖的资源已经发生了冲突，迁移资源必须同时拷贝资源依赖项，所以`Assets/MaterialLibrary/M_Master`就被莫名其妙覆盖了。

和这种情况类似，任何资源的依赖项的不兼容都会让资源在迁移中被破坏掉，如果没有资源顶级目录，资源迁移就会变成一场非常让人恶心的任务。

#### [](#223-范例模板以及商场中的资源都是安全没有风险的)2.2.3 范例，模板以及商场中的资源都是安全没有风险的

正如上面 [2.2.2](#2.2.2) 所讲，如果一个团队想把官方范例、模板以及商城中购买的资源放到自己的工程中，那么这些资源都是可以保证不会干扰现有工程的，除非你购买的资源工程和你的工程同名。

当然也不能完全信任商城上的资源能够完全遵守[顶级目录规则](#2.2)。的确有一些商城资源，尽管大部分资源放在了顶级目录下面，但仍然留下了部分资源污染了`Assets`目录

如果坚持这个原则 [2.2](#2.2)，最糟糕的情况就是购买了两个不同的商场资源，结果发现他们使用了相同的 Unity 标准资源。但只要你坚持把自己的资源放在自己的工程目录中，并且把使用的 EPIC 示例资源也放在自己的目录中，那么自己工程也不会受到影响。

#### [](#224-容易维护dlc子工程以及补丁包)2.2.4 容易维护 DLC、子工程、以及补丁包

如果你的工程打算开发 DLC 或者子工程，那么这些子工程所需要的资源应该迁移出来放在另一个顶级目录中，这样的结构使得编译这些版本时可以区别对待子工程中的资源。子工程中的资源的迁入和迁出代价也更小。如果你想在子项目中修改一些原有工程中的资源，那么可以把这些资源迁移到子工程目录中，这样不会破坏原有工程。

### [](#23-用来做临时测试的开发者目录-)2.3 用来做临时测试的开发者目录 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

在一个项目的开发期间，团队成员经常会有一个'沙箱'目录用来做测试而不会影响到工程本身。因为工作是连续的，所以即使这些'沙箱'目录也需要上传到源码服务器上保存。但并不是所有团队成员都需要这种开发者目录的，但使用开发者目录的成员来说，一旦这些目录是在服务器上管理的，总会需要一些麻烦事。

首先团队成员极容易使用这些尚未准备好的资源，这些资源一旦被删除就会引发问题。例如一个做模型的美术正在调整一个模型资源，这时一个做场景编辑的美术如果在场景中使用了这个模型，那么很可能会导致莫名其妙的问题，进而造成大量的工作浪费。

但如果这些模型放在开发者目录中，那么场景美术人员就没有任何理由使用这些资源。

一旦这些资源真正准备好，那么开发人员应该把它们移到正式的工程目录中并修复引用关系，这实际上是让资源从实验阶段'推进'到了生产阶段。

### [](#24-所有的地图文件应该保存在一个名为scenes的目录中-)2.4 所有的地图文件应该保存在一个名为'Scenes'的目录中 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

地图文件非常特殊，几乎所有工程都有自己的一套关于地图的命名规则，尤其是使用了 sub-levels 或者 streaming levels 技术时。但不管你如何组织自己的命名规则，都应该把所有地图保存在`/Assets/ProjectName/Scenes`

记住，尽量使用不浪费大家的时间的方法去解释你的地图如何打开。比如通过子目录的方法去组织地图资源，例如建立 `Scenes/Campaign1/` 或 `Scenes/Arenas`，但最重要的是一定要都放在`/Assets/Project/Scenes`

这也有助于产品的打版本工作，如果工程里的地图保存的到处都是，版本工程师还要到处去找，就让人很恼火了，而把地图放在一个地方，做版本时就很难漏掉某个地图，对于烘培光照贴图或者质量检查都有利。

### [](#25-使用core目录存储核心代码资源以及其他文件资源-)2.5 使用`Core`目录存储核心代码资源以及其他文件资源 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

使用`/Assets/Project/Core`这个目录用来保存一个工程中最为核心的资源。例如，非常基础的`Manager`, `Character`, `PlayerController`, `Sington`, `PlayerState`，以及如此相关的一些资源也应该放在这里。

这个目录非常明显的告诉其他团队成员:"不要碰我！"。非引擎程序员很少有理由去碰这个目录，如果工程目录结构合理，那么游戏设计师只需要使用子类提供的功能就可以工作，负责场景编辑的员工只需要使用专用的 Prefab 就可以，而不用碰到这些基础类。

例如，如果项目需要设计一种可以放置在场景中并且可以被捡起的物体，那么应该首先设计一个具有被捡起功能的基类放在`Core/Pickups`目录中，而各种具体的可以被捡起的物体诸如药瓶、子弹这样的物体，应该放在`/Assets/Project/Placeables/Pickups/`这样的目录中。游戏设计师可以在这些目录中定义和设计这些物体，所以他们不应该去碰`Core/Pickups`目录下的代码，要不然可能无意中破坏工程中的其他功能

### [](#26-不要创建名为assets-或者-assettypes的目录-)2.6 不要创建名为`Assets` 或者 `AssetTypes`的目录 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

#### [](#261-创建一个名为assets的目录是多余的-)2.6.1 创建一个名为`Assets`的目录是多余的。 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

因为本来所有目录就是用来保存资源的

#### [](#262-创建名为meshes-textures或者materials的目录是多余的-)2.6.2 创建名为`Meshes`、 `Textures`或者`Materials`的目录是多余的。 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

资源的文件名本身已经提供了资源类型信息，所以在目录名中再提供资源类型信息就是多余了，而且使用资源浏览器的过滤功能，可以非常便利的提供相同的功能。

比如想查看`Environment/Rocks/`目录下所有的静态 Mesh 资源？只要打开静态 Mesh 过滤器就可以了，如果所有资源的文件名已经正确命名，这些文件还会按照文件名和前缀正确排序，如果想查看所有静态 Mesh 和带有骨骼的 Mesh 资源，只要打开这两个过滤器就可以了，这种方法要比通过打开关闭文件夹省事多了。

> 这种方法也能够节省路径长度，因为用前缀`S_`只有两个字符，要比使用`Meshes/`七个字符短多了。

这么做其实也能防止有人把 Mesh 或者纹理放在`Materials`目录这种愚蠢行为。

### [](#27-超大资源要有自己的目录结构-)2.7 超大资源要有自己的目录结构 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

这节可以视为针对 [2.6](#2.6) 的补充

有一些资源类型通常文件数量巨大，而且每个作用都不同。典型的例子是动画资源和声音资源。如果你发现有 15 个以上的资源属于同一个逻辑类型，那么它们应该被放在一起。

举例来说，角色共用的动画资源应该放在`Characters/Common/Animations`目录中，并且其中应该还有诸如`Locomotion` 或者`Cinematic`的子目录

> 这并不适用与纹理和材质。比如`Rocks`目录通常会包含数量非常多的纹理，但每个纹理都都是属于特定的石头的，它们应该被正确命名就足够了。即使这些纹理属于[材质库](#2.8)

### [](#28-材质库materiallibrary-)2.8 材质库`MaterialLibrary` [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

如果你的工程中使用了任何基础材质、分层材质，或者任何被重复使用而不属于特定模型的材质和纹理，这些资源应该放在材质库目录`Assets/Project/MaterialLibrary`。

这样可以很容易管理这些'全局'材质

> 这也使得'只是用材质实例'这个原则得以执行的比较容易。如果所有的美术人员都只是用材质实例，那么所有的原始材质都应该保存在这个目录中。你可以通过搜索所有不在`MaterialLibrary`中的基础材质来验证这一点。

`MaterialLibrary`这个目录并不是仅能保存材质资源，一些共用的工具纹理、材质函数以及其他具有类似属性的资源都应该放在这个目录或子目录中。例如，噪声纹理应该保存在`MaterialLibrary/Utility`目录中。

任何用来测试或调试的材质应该保存在`MaterialLibrary/Debug`中，这样当工程正式发布时，可以很容易把这些材质从删除，因为这些材质如果不删除，可能在最终产品中非常扎眼。

[](#3-代码-)3. 代码 ![#]
--------------------

### [](#目录-3)目录

> 3.1 编译 [Compiling](#bp-compiling)

> 3.2 变量 [Variables](#bp-vars)

> 3.3 函数 [Functions](#bp-functions)

### [](#31-编译-)3.1 编译 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

需要保证所有代码在编译时 0 警告和 0 错误。你应该尽快修复所有警告和异常，以免它们造成可怕的麻烦。

_绝对不要_提交那些测试的代码，如果你需要通过源码服务器保存，那么必须暂时防止到 Developer 目录下。

### [](#32-变量-)3.2 变量 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

变量 (`variable`) 和属性 (`property`) 这两个词经常是可以互换的。

#### [](#目录-4)目录

> 3.2.1 命名 [Naming](#bp-vars)

> 3.2.2 可编辑 [Editable](#bp-vars-editable)

> 3.2.3 分类 [Categories](#bp-vars-categories)

> 3.2.4 权限 [Access](#bp-vars-access)

#### [](#321-命名规范-)3.2.1 命名规范 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

##### [](#3211-使用名词-)3.2.1.1 使用名词 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

所有非布尔类型的变量必须使用简短、清晰并且意义明确的名词作为变量名。

##### [](#3212-pascalcase-)3.2.1.2 PascalCase [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

所有非布尔类型的变量的大小写需要遵守 [PascalCase](#terms-cases) 规则。

###### [](#3212e-范例)3.2.1.2e 范例:

*   `Score`
*   `Kills`
*   `TargetPlayer`
*   `Range`
*   `CrosshairColor`
*   `AbilityID`

##### [](#3213-布尔变量需要前缀-b--)3.2.1.3 布尔变量需要前缀 `b` [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

所有布尔类型变量需要遵守 [PascalCase](#terms-cases) 规则，但前面需要增加小写的`b`做前缀。

例如: 用 `bDead` 和 `bEvil`, **不要** 使用`Dead` 和 `Evil`.

##### [](#3214-布尔类型变量命名规则-)3.2.1.4 布尔类型变量命名规则 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

###### [](#32141-一般的独立信息-)3.2.1.4.1 一般的独立信息 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

布尔类型变量如果用来表示一般的信息，名字应该使用描述性的单词，不要包含具有提问含义的词汇，比如`Is`，这个词是保留单词。

例如：使用`bDead` and `bHostile`，**不要**使用`bIsDead` and `bIsHostile`。

也不要使用类似于`bRunning`这样的动词，动词会让含义变得复杂。

###### [](#32142-复杂状态-)3.2.1.4.2 复杂状态 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

不要使用布尔变量保存复杂的，或者需要依赖其他属性的状态信息，这会让状态变得复杂和难以理解，如果需要尽量使用枚举来代替。

例如：当定义一个武器时，**不要**使用`bReloading` 和 `bEquipping`这样的变量，因为一把武器不可能即在 reloading 状态又在 equipping 状态，所以应该使用定义一个叫做`EWeaponState`的枚举，然后用一个枚举变量`WeaponState`来代替，这也使得以后增加新的状态更加容易。

例如：**不要**使用`bRunning`这样的变量，因为你以后有可能还会增加`bWalking` 或者 `bSprinting`，这也应该使用一个枚举来非常清晰的定义这样的状态。

##### [](#3215-考虑上下文-)3.2.1.5 考虑上下文 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

蓝图中的变量命名时需要考虑上下文环境，避免重复不必要的定义。

###### [](#3215e-例如)3.2.1.5e 例如:

假设有一个类名为 `PlayerCharacter`.

**不好的命名**

*   `PlayerScore`
*   `PlayerKills`
*   `MyTargetPlayer`
*   `MyCharacterName`
*   `CharacterSkills`
*   `ChosenCharacterSkin`

这些变量的命名都很臃肿。因为这些变量都是属于一个角色类`layerCharacter`的，没必要在变量中再重复这一点。

**好的命名**

*   `Score`
*   `Kills`
*   `TargetPlayer`
*   `Name`
*   `Skills`
*   `Skin`

##### [](#3216-不要在变量中包含原生变量类型名-)3.2.1.6 **不要**在变量中包含原生变量类型名 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

所谓原生变量是指那些最简单的保存数据的变量类型，比如布尔类型、整数、浮点数以及枚举。

原生类型的变量名中不应该包含变量类型名。

例如：使用`Score`, `Kills`, 以及 `Description`，**不要**使用`ScoreFloat`, `FloatKills`, `DescriptionString`。

但也有例外情况，当变量的含义包含了 "多少个" 这样的信息，**并且**仅用一个名字无法清晰的表达出这个含义时。

比如：游戏中一个围墙生成器，需要有一个变量保存在 X 轴上的生成数量，那么需要使用`NumPosts` 或者 `PostsCount`这样的变量，因为仅仅使用`Posts`可能被误解为某个保存 Post 的数组

##### [](#3217-非原生类型的变量需要包含变量类型名-)3.2.1.7 非原生类型的变量，需要包含变量类型名 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

非原生类型的变量是指那些通过数据结构保存一批原生类型的复杂变量类型，比如 Structs、Classes、Interface，还有一些有类似行为的原生变量比如`Text` 和 `Name`也属于此列。

> 如果仅仅是原生变量组成的数组，那么这个数组仍然属于原生类型

这些变量的名字应该包含数据类型名，但同时要考虑不要重复上下文。

如果一个类中包拥有一个复杂变量的实例，比如一个`PlayerCharacter`中有另一个变量`Hat`，那么这个变量的名字就不需要包含变量类型了。

例如: 使用 `Hat`、`Flag`以及 `Ability`，**不要**使用`MyHat`、`MyFlag` 和 `PlayerAbility`

但是，如果一个类并不拥有这个属性，那么就需要在这个属性的名字中包含有类型的名字了

例如：一个类`Turret`用来顶一个炮塔，它拥有瞄准`PlayerCharacter`作为目标的能力，那么它内部会保存一个变量作为目标，名字应该是`TargetPlayer`，这个名字非常清楚的指明了这个变量的数据类型是什么。

##### [](#3218-数组-)3.2.1.8 数组 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

数组的命名规则通常和所包含的元素的规则一样，但注意要用复数。

例如：用`Targets`、`Hats`以及 `EnemyPlayers`，**不要**使用`TargetList`、`HatArray` 或者 `EnemyPlayerArray`

#### [](#322-可编辑变量-)3.2.2 可编辑变量 [![](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)](https://camo.githubusercontent.com/5610bfb0f1a5242b11ab815e3366edc9a649ff91ac9a1396b83c6c420bbed37c/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d7061727469616c5f737570706f72742d79656c6c6f772e737667)

所有可以安全的更改数据内容的变量都需要被标记为`public`

相反，所有不能更改或者不能暴露给设计师的变量都需要标记为`private`，如果确实要暴露给设计师但又不想破坏程序封装，这些变量就需要被标为`[SerializedFeild]`

总之不要轻易使用`public`。

##### [](#3221-tooltips-)3.2.2.1 Tooltips [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

对于所有标记为`public`的变量，包括被标记为 `[SerializedFeild]`的变量，都应该加上`[Tooltip("说明内容")]`内填写关于如何改变变量值，以及会产生何种效果的说明。

##### [](#3222-滑动条slider以及取值范围-)3.2.2.2 滑动条 (Slider) 以及取值范围 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

对于可编辑的变量，如果不适合直接输入具体数值，那么应该通过一个滑动条 (Slider) 并且加上取值范围来让设计师输入。可以用`[Range(min,max)]`属性实现。

举例：一个产生围墙的类，拥有一个`PostsCount`的变量，那么 - 1 显然适合不合理的输入，所以需要设上取值范围注明 0 是最小值，

一个变量的取值范围只有当明确知道其范围时才需要定义，因为滑块的取值范围的确能够阻止用户输入危险数值，但用户仍然能够通过手动输入的方式输入一个超出滑块范围的值给变量，如果变量的取值范围未定义，那么这个值就会变得'很危险'但还是在合理的。

#### [](#323-分类-)3.2.3 分类 [![](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)](https://camo.githubusercontent.com/24207926b2d527121c41214763714d2d030cdc8311cb07a235dfa7a05fa75a2d/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d737570706f727465642d677265656e2e737667)

如果一个类的变量很少，那么没有必要使用分类

如果一个类的变量规模达到中等 (5-10)，那么所有`public`的变量应该自己的分类, 可以使用`[Header("HealthSettings")]`这样的属性实现。

举例：一个武器的类中的变量分类目录大致如下：

```
|-- Config
|	|-- Animations
|	|-- Effects
|	|-- Audio
|	|-- Recoil
|	|-- Timings
|-- Animations
|-- State
|-- Visuals


```

##### [](#3241-私有变量-)3.2.4.1 私有变量 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

尽量不要把变量声明为 private 类型，除非变量一开始就打算永远被类内部访问，并且类本身也没打算被继承。尽量用`protected`，private 类型用在当你有非常清楚的理由要去限制子类的能力。

### [](#33-函数事件以及事件派发器-)3.3 函数、事件以及事件派发器 [![](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)](https://camo.githubusercontent.com/4cf6fb338a32e0c605795ae5057cbae643003902e6877bf069f33f5470a930d5/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f6c696e742d756e737570706f727465642d7265642e737667)

这一节用来解释应该如何管理函数、事件以及事件派发器。除非特殊说明，所有适用于函数的规则，同样适用于事件。

#### [](#331-函数命名)3.3.1 函数命名

对于函数、事件以及事件派发器的命名极其重要，仅仅从一个名字本身，就有很多条件要考虑，比如说：

*   是纯虚函数吗？?
*   是状态查询函数吗?
*   是事件相应函数吗?
*   是远程调用函数吗?
*   函数的目的是什么？

如果命名得当，这些问题甚至更多问题的答案会在名字中体现出来。

#### [](#3311-所有函数的命名都应该是动词)3.3.1.1 所有函数的命名都应该是动词

所有函数和事件执行者都是需要做一些动作，可能是去获取信息，也可能是数据计算，或者搞点什么事情。因此，所有函数都应该用动词开始，并且用一般现代时态，并且有上下文来表明它们究竟在做什么

好的例子:

*   `Fire` - 如果类是一个角色或者武器，那么这是一个好命名，如果是木桶，玻璃，那这个函数就会让人困惑了。
*   `Jump` - 如果类是一个角色，那么这是个好名字，如果不是，那么需要一些上下文来解释这个函数的含义
*   `Explode`
*   `ReceiveMessage`
*   `SortPlayerArray`
*   `GetArmOffset`
*   `GetCoordinates`
*   `UpdateTransforms`
*   `EnableBigHeadMode`
*   `IsEnemy` - ["Is" 是个动词](http://writingexplained.org/is-is-a-verb)

不好的例子:

*   `Dead` - 是已经死了？还是死的动作?
*   `Rock`
*   `ProcessData` - 无意义，这个名字等于没说.
*   `PlayerState` - 不能用名词
*   `Color` - 如果是动词，那么缺少上下文，如果是名词，也不行.

#### [](#3313-返回布尔变量的信息查询函数应该是问询函数)3.3.1.3 返回布尔变量的信息查询函数应该是问询函数

如果一个函数不改变类的状态，并且只是返回信息、状态或者计算返回给调用者 yes/no，这应该是一个问询函数。同样遵守[动词规则](#bp-funcs-naming-verbs)。

非常重要的是，应该假定这样的函数其实就是执行某种动作，并且返回动作是否执行成功。

好的例子:

*   `IsDead`
*   `IsOnFire`
*   `IsAlive`
*   `IsSpeaking`
*   `IsHavingAnExistentialCrisis`
*   `IsVisible`
*   `HasWeapon` - ["Has" 是动词.](http://grammar.yourdictionary.com/parts-of-speech/verbs/Helping-Verbs.html)
*   `WasCharging` - ["Was" 是动词 "be" 的过去式](http://grammar.yourdictionary.com/parts-of-speech/verbs/Helping-Verbs.html) 用 "was" 表示查询以前的状态
*   `CanReload` - ["Can" 是动词](http://grammar.yourdictionary.com/parts-of-speech/verbs/Helping-Verbs.html)

坏的例子:

*   `Fire` - 是查询正在开火？还是查询能不能开火？
*   `OnFire` - 有可能和事件派发器函数混淆
*   `Dead` - 是查询已经死亡？还是查询会不会死亡？
*   `Visibility` - 是查询可见状态？还是设置可见状态？

#### [](#3314-事件的响应函数和派发函数都应该以on开头)3.3.1.4 事件的响应函数和派发函数都应该以`On`开头

事件的响应函数和派发函数都应该以`On`开头，然后遵守[动词规则](#bp-funcs-naming-verbs)，如果是过去式，那么动词应该移到最后以方便阅读

在遵守动词规则的时候，需要优先考虑英语中的[固定句式](http://dictionary.cambridge.org/us/grammar/british-grammar/about-words-clauses-and-sentences/collocation)

有一些系统用`Handle`来表示事件响应，但在'Unreal'用的是`On`而不是`Handle`，

好的例子:

*   `OnDeath` - 游戏中非常常见
*   `OnPickup`
*   `OnReceiveMessage`
*   `OnMessageRecieved`
*   `OnTargetChanged`
*   `OnClick`
*   `OnLeave`

坏的例子:

*   `OnData`
*   `OnTarget`
*   `HandleMessage`
*   `HandleDeath`

#### [](#3315-远程调用函数应该用目标作为前缀)3.3.1.5 远程调用函数应该用目标作为前缀

任何时候创建 RPC 函数，都应该把目标作为前缀放在前面，例如`Server`、 `Client`或者 `Multicast`，没有例外。

前缀之后的部分，遵守上面的其他规则。

好的例子:

*   `ServerFireWeapon`
*   `ClientNotifyDeath`
*   `MulticastSpawnTracerEffect`

坏的例子:

*   `FireWeapon` - 没有使用目标前缀
*   `ServerClientBroadcast` - 混淆.
*   `AllNotifyDeath` - 用 `Multicast`, 不要用 `All`.
*   `ClientWeapon` - 没有用动词, 让人困惑.

#### [](#332-所有函数都应该有返回节点)3.3.2 所有函数都应该有返回节点

所有函数都应该有返回节点，没有例外。

比如说，有程序员在并行序列中添加了一个新的分支，或者在循环体外添加逻辑但没有考虑到循环中的意外返回，那么这些情况都会造成程序的执行序列出现意外。

[](#)};
=======