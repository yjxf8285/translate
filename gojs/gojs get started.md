GoJS是一个实现交互类图表（比如流程图，树图，关系图，力导图等等）的JS库。本文将介绍GoJs的精华部分。
因为GoJS依赖于HTML5，所以请保证您的浏览器版本支持HTML5，当然还要加载这个库。

<pre class='prettyprint'>
<!DOCTYPE html>  <!-- HTML5 文档类型 -->
<html>
<head>
  <!-- 调式或开发模式下请使用 go-debug.js -->
  <script src="go-debug.js"></script>
  . . .
</pre>
您可以在 https://gojs.net/latest/doc/download.html 下载GoJS以及所有的示例。或者使用下面的CDNJS直接引入：

<pre class='prettyprint'>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gojs/1.8.13/go-debug.js"></script>
</pre>
每个GoJS图表必须包含在一个指定宽高的div容器内
<pre class='prettyprint'>
<!-- 这个DIV必须指定宽高，否者不会被渲染出来
     我们通常为DIV设置一个背景颜色以便于我们更便捷的观察 -->
<div id="myDiagramDiv"
     style="width:400px; height:150px; background-color: #DAE4E4;"></div>
</pre>
在JS代码部分，您需要传入一个div的id作为参数来创建一个图表
<pre class='prettyprint'>
var $ = go.GraphObject.make;
var myDiagram =
  $(go.Diagram, "myDiagramDiv");
</pre>
就这样创建了一个空的图表
[图1]
注意所有GoJS的属性和方法都在go这个命名空间下。所有GoJS的类名，例如Diagram、Node、Panel、 Shape、TextBlock也都使用go作为前缀

本文将介绍如何使用go.GraphObject.make来创建一个GoJS对象，更多细节请阅读 Building Objects in GoJS（https://gojs.net/latest/intro/buildingObjects.html）。为了更方便的使用go.GraphObject.make对象，我们通常将它赋值给一个例如$的缩写变量。当然如果$被其他库(jquery等等)占用了，您也可以使用$$或MAKE或GO。

# Diagrams（图表） 和 Models（模型）

Diagrams中的Nodes（节点）和Links（连线）呈现是由Model进行管理的。GoJs使用model-view（MV架构）的模式,Models作为数据层来管理那些描述性的数据（JS数组对象），Diagrams则负责视图层，将Nodes和Links的数据以可视化的方式渲染出来。注意我们编程后操作的只针对于Model的数据层，而不是Diagrams的视图层。你可以按照需求在Models数据对象上面添加任意属性，但是请不要修改Diagram的prototype（原型）和GraphObject（绘图单元）的classes（类）.
<pre class='prettyprint'>
var $ = go.GraphObject.make;
var myDiagram =
  $(go.Diagram, "myDiagramDiv",
    {
      initialContentAlignment: go.Spot.Center, // 居中显示Diagram内容
      "undoManager.isEnabled": true // 打开Ctrl-Z撤销和Ctrl-Y重做功能
    });

var myModel = $(go.Model);
// 在model的数据中, 每个节点数据的值都是由一个JS对象来表示:
myModel.nodeDataArray = [
  { key: "Alpha" },
  { key: "Beta" },
  { key: "Gamma" }
];
myDiagram.model = myModel;
</pre>
[图2]

上面图表中通过Model数据的3个节点数据显示了3个Nodes，同时它还支持一些交互：
* 点击并拖动背景可以平移视图
* 点击可以选中一个节点，也可以拖拽它
* 在背景处点击并拖拽出创建一个选择区域来框选节点
* 使用CTRL-C和CTRL-V复制粘贴节点，或者按住Ctrl点击节点来进行多选
* 按下Delete键删除已选中的节点
* 如果undoManager.isEnabled的参数配置为true,就可以使用Ctrl-Z来撤销和Ctrl-Y重做你的操作

# Nodes样式
Nodes样式的模板由GraphObjects对象和它的参数配置组成。创建一个Nodes时，有几个预设的building block类供我们使用：

* Shape 预定义的或者自定义的几何图形
* TextBlock 拥有各种各样字体的文本（可编辑）
* Picture 图片
* Panel 根据不同面板的类型，它可以包含其他位置或是尺寸不同的对象。(列如表格、 竖形列表和拉伸容器等)

所有的这些building block类都是由GraphObjects抽象对象衍生出来。所以我们也可以把它们叫做GraphObjects或objects或者elements。因为GraphObject不是DOM元素，所以创建和修改它们对性能开销不大。

我们实现了数据绑定，通过监听Model数据，自动改变Nodes上的GraphObjects外观和行为。Model数据对象是一个普通的JavaScript对象。您可以使用任何字段名为Model中的Nodes数据的key命名。

默认的Nodes模板非常简单:仅仅是包含一个TextBlock的Nodes。TextBlock的text字段和Model中Nodes数据的key字段绑定在一起。在代码中，模板看起来是这样的:
<pre class='prettyprint'>
myDiagram.nodeTemplate =
  $(go.Node,
    $(go.TextBlock,
      // 把TextBlock.text 绑在 Node.data.key上
      new go.Binding("text", "key"))
  );
</pre>
注意这里没有Node.key这个字段. 但是你可以从someNode.data.key获取它.  

TextBlock、Shapes和Pictures是GoJS原生的building blocks。TextBlocks不能包含图片;Shapes不能包含文字。如果你想让你的Node显示文字，你必须使用TextBlocks。如果你想绘制一些几何图形，你就必须使用Shape。

更常用的Nodes模板结构类似这样
<pre class='prettyprint'>
myDiagram.nodeTemplate =
  $(go.Node, "Vertical", // 第2个参数是Panel的类型。（译者注：比如横向排列还是纵向排列）
    /* Node参数配置 */
    { // Node.location 定位的基准点设置在每个节点的中心处
      locationSpot: go.Spot.Center
    },

    /* 这里添加数据绑定 */
    // 例如 Node.location的参数值绑在 Node.data.loc
    new go.Binding("location", "loc"),

    /* 在节点中添加 GraphObjects */
    // 这个Shape将垂直放在TextBlock上
    $(go.Shape,
      "RoundedRectangle", // 这个参数可以指定一个预定义的图形（圆角矩形）。
      { /* 为Shape设置配置项 */ },
      // 例如Shape的Shape.figure参数值绑定在Node.data.fig
      new go.Binding("figure", "fig")),

    $(go.TextBlock,
      "default text",  // 这个参数初始化了一个默认的文本
      { /* 配置TextBlock */ },
      // 例如 TextBlock.text参数的值绑定在Node.data.key
      new go.Binding("text", "key"))
  );
</pre>
通常，Panels内可以定义任意数量的GraphObjects，并且每个类都有自己的参数配置。
那么现在我们已经知道了如何创建一个Node模板，来看一个实际的例子。我们要在常见的组织图中创建一个简单的模板 —— 一个图片跟着一个名称。让们我们分析一下这个Note模板：
* 一个面板类型为“水平”的节点，也就是说它由两个元素水平排列组和而成：
* 一个作为头像使用的Picture，与数据中的source字段绑定
* 一个作为名字使用的TextBlock，与数据中的text字段绑定
<pre class='prettyprint'>
var $ = go.GraphObject.make;
var myDiagram =
  $(go.Diagram, "myDiagramDiv",
    {
      initialContentAlignment: go.Spot.Center, // 居中显示
      "undoManager.isEnabled": true // 支持 Ctrl-Z 和 Ctrl-Y 操作
    });

// 定义个简单的 Node 模板
myDiagram.nodeTemplate =
  $(go.Node, "Horizontal",
    // 为整个Node背景设置为浅蓝色
    { background: "#44CCFF" },
    $(go.Picture,
      // Pictures 应该指定宽高.
      // 当没有图片时显示红色的背景
      // 或者当图片为透明的时候也是.
      { margin: 10, width: 50, height: 50, background: "red" },
      // Picture.source参数值与模型数据中的"source"字段绑定
      new go.Binding("source")),
    $(go.TextBlock,
      "Default Text",  // 初始化默认文本
      // 文字周围的空隙, 大号字体, 白色笔画:
      { margin: 12, stroke: "white", font: "bold 16px sans-serif" },
      // TextBlock.text参数值与模型数据中的"name"字段绑定
      new go.Binding("text", "name"))
  );

var model = $(go.Model);
model.nodeDataArray =
[ // 注意每个节点数据对象内掌握着那些我们需要的参数;
  // 这里我们添加了 "name" 和 "source" 参数
  { name: "Don Meow", source: "cat1.png" },
  { name: "Copricat", source: "cat2.png" },
  { name: "Demeter",  source: "cat3.png" },
  { /* 空节点数据 */  }
];
myDiagram.model = model;
</pre>
这是代码生成后的图表
[图3]

当数据中的图片没有加载或者名称未知的时候，我们可以为节点提供一个默认的显示状态。就如那个空节点那样，完美的呈现了数据绑定到空对象时的样子。

# Models的种类

自定义的node模板让我们的图表看起来更美观，但是我们还要展示更多东西。或许是一个用来证明Don Meow是一个企业的老板的关系图。那么好，我创建一个完整的关系图，通过添加一些连线来表示这些独立的节点之间的对应关系，同时这些节点能够进行自动定位和排版。

在我们的图表里为了得到这些连线，基本的Model已经满足不了需求。我们必须从GOJS中的支持连线的另外两个Models里选择1个，他们是GraphLinksModel和TreeModel（详见https://gojs.net/latest/intro/usingModels.html）。

GraphLinksModel中，除了model.nodeDataArray还有model.linkDataArray。它包含一个数组对象，通过"to"和"from"来描述没一个连线。这里有一个节点A链接到节点B，节点B链接到节点C的例子：
<pre class='prettyprint'>
var model = $(go.GraphLinksModel);
model.nodeDataArray =
[
  { key: "A" },
  { key: "B" },
  { key: "C" }
];
model.linkDataArray =
[
  { from: "A", to: "B" },
  { from: "B", to: "C" }
];
myDiagram.model = model;
</pre>
GraphLinksModel允许两个节点之间存在任何数量和任意方向的连线。比如A到B可以连10条线，B到A可以连3条以上反方向的线。

TreeModel的实现方式略有不同。数组内不再存放各个连线的数据，而是通过具体的父级节点的方式来创建那些连线数据的，然后再从这些父子关系中创建出连线。这有一个树形结构的简单例子，A连B和B连C：
<pre class='prettyprint'>
var model = $(go.TreeModel);
model.nodeDataArray =
[
  { key: "A" },
  { key: "B", parent: "A" },
  { key: "C", parent: "B" }
];
myDiagram.model = model;
</pre>
TreeModel比GraphLinksModel更简单，但是不能随意的建利连接关系，就像2个节点之间有多条线，又或者有多个父级节点。我们的关系图是一个简单的分等级树形结构，所以我们就选择TreeModel来举例子。

首先，我们将添加更多的节点数据来完成这个数据，使用TreeModel，在数据中指定键值和父级。
<pre class='prettyprint'>
var $ = go.GraphObject.make;
var myDiagram =
  $(go.Diagram, "myDiagramDiv",
    {
      initialContentAlignment: go.Spot.Center, // 居中显示内容
      "undoManager.isEnabled": true // 打开 Ctrl-Z 和 Ctrl-Y 撤销重做功能
    });

// 我们早先定义的节点模板
myDiagram.nodeTemplate =
  $(go.Node, "Horizontal",
    { background: "#44CCFF" },
    $(go.Picture,
      { margin: 10, width: 50, height: 50, background: "red" },
      new go.Binding("source")),
    $(go.TextBlock, "Default Text",
      { margin: 12, stroke: "white", font: "bold 16px sans-serif" },
      new go.Binding("text", "name"))
  );

var model = $(go.TreeModel);
model.nodeDataArray =
[ // 必须有"key"和"parent"的字段名,
  // 你还可以添加任何需要的其他字段
  { key: "1",              name: "Don Meow",   source: "cat1.png" },
  { key: "2", parent: "1", name: "Demeter",    source: "cat2.png" },
  { key: "3", parent: "1", name: "Copricat",   source: "cat3.png" },
  { key: "4", parent: "3", name: "Jellylorum", source: "cat4.png" },
  { key: "5", parent: "3", name: "Alonzo",     source: "cat5.png" },
  { key: "6", parent: "2", name: "Munkustrap", source: "cat6.png" }
];
myDiagram.model = model;
</pre>
# 图表布局

如你所见，TreeModel把那些节点的自动的连接在了一起。但是不能方便的显示等级对应关系。

在节点没有指定坐标的时候，图表会显示一个用网格形式排列的默认布局。我们可以为每个节点指定一个坐标来解决这个混乱的组织结构，但是有个更简单的办法，我们可以使用一个能够自动定位的布局方案。

我们想看到使用TreeModel实现的层级关系，所以最自然的布局就是TreeLayout。TreeLayout默认是从左向右排列，所以我们需要把设置为从上到下的形式（常见关系图布局形式）， 我们将角度值设置为90.

GoJs中使用布局很简单。每种布局都有很多属性去影响最终的显示效果。这是一个简单的树形布局示例及配置参数。
<pre class='prettyprint'>
// 定义一个由上到下的树形结构
myDiagram.layout =
  $(go.TreeLayout,
    { angle: 90, layerSpacing: 35 });
</pre>
GoJs还有几个其他的布局，见https://gojs.net/latest/intro/layouts.html

我们看一下给图表添加了布局后的结果：

<pre class='prettyprint'>
var $ = go.GraphObject.make;
var myDiagram =
  $(go.Diagram, "myDiagramDiv",
    {
      initialContentAlignment: go.Spot.Center, // 居中显示内容
      "undoManager.isEnabled": true, // 打开 Ctrl-Z 和 Ctrl-Y 撤销重做功能
      layout: $(go.TreeLayout, // 1个特殊的树形排列 Diagram.layout布局
                { angle: 90, layerSpacing: 35 })
    });

// 我们早先定义的模板
myDiagram.nodeTemplate =
  $(go.Node, "Horizontal",
    { background: "#44CCFF" },
    $(go.Picture,
      { margin: 10, width: 50, height: 50, background: "red" },
      new go.Binding("source")),
    $(go.TextBlock, "Default Text",
      { margin: 12, stroke: "white", font: "bold 16px sans-serif" },
      new go.Binding("text", "name"))
  );

var model = $(go.TreeModel);
model.nodeDataArray =
[
  { key: "1",              name: "Don Meow",   source: "cat1.png" },
  { key: "2", parent: "1", name: "Demeter",    source: "cat2.png" },
  { key: "3", parent: "1", name: "Copricat",   source: "cat3.png" },
  { key: "4", parent: "3", name: "Jellylorum", source: "cat4.png" },
  { key: "5", parent: "3", name: "Alonzo",     source: "cat5.png" },
  { key: "6", parent: "2", name: "Munkustrap", source: "cat6.png" }
];
myDiagram.model = model;

</pre>

# 连线模板

接下来我们构造一个新的连线模板,它要更好地适应各种boxy节点。连线与节点是不同的两个部分，连线的主要元素是一个被GoJS动态创建的几何形状。我们的连线就是由这些形状组成的，它的笔触比默认的要粗一些。而且不像默认模板那样，我们不需要箭头。并且我们把默认的路由属性改成直角的方式，再给一个角度值让它有一个圆角。

<pre class='prettyprint'>
// 定义一个直角路由形式的连线模板, 去掉箭头
myDiagram.linkTemplate =
  $(go.Link,
    // 默认的路由 go.Link.Normal
    // 默认角度值 0
    { routing: go.Link.Orthogonal, corner: 5 },
    $(go.Shape, { strokeWidth: 3, stroke: "#555" }) // 线的宽度和笔画的颜色

    // 如果我们要显示箭头，就应该定义一个有箭头的形状
    // $(go.Shape, { toArrow: "Standard", stroke: null }
    );
</pre>

Combining our Link template with our Node template, TreeModel, and TreeLayout, we finally have a full organization diagram. The complete code is repeated below, and the resulting diagram follows:

结合连线模板和节点模板，TreeModel和TreeLayout，我们最终完成了一个完整的关系图，下面是完整的代码和生成后的效果：
<pre class='prettyprint'>
var $ = go.GraphObject.make;

var myDiagram =
  $(go.Diagram, "myDiagramDiv",
    {
      initialContentAlignment: go.Spot.Center, // 居中显示内容
      "undoManager.isEnabled": true, // 打开 Ctrl-Z 和 Ctrl-Y 撤销重做功能
      layout: $(go.TreeLayout, // 1个特殊的树形排列 Diagram.layout布局
                { angle: 90, layerSpacing: 35 })
    });

// 我们早先定义的模板
myDiagram.nodeTemplate =
  $(go.Node, "Horizontal",
    { background: "#44CCFF" },
    $(go.Picture,
      { margin: 10, width: 50, height: 50, background: "red" },
      new go.Binding("source")),
    $(go.TextBlock, "Default Text",
      { margin: 12, stroke: "white", font: "bold 16px sans-serif" },
      new go.Binding("text", "name"))
  );

// 定义一个直角路由形式的连线模板, 去掉箭头
myDiagram.linkTemplate =
  $(go.Link,
    { routing: go.Link.Orthogonal, corner: 5 },
    $(go.Shape, { strokeWidth: 3, stroke: "#555" })); // the link shape

var model = $(go.TreeModel);
model.nodeDataArray =
[
  { key: "1",              name: "Don Meow",   source: "cat1.png" },
  { key: "2", parent: "1", name: "Demeter",    source: "cat2.png" },
  { key: "3", parent: "1", name: "Copricat",   source: "cat3.png" },
  { key: "4", parent: "3", name: "Jellylorum", source: "cat4.png" },
  { key: "5", parent: "3", name: "Alonzo",     source: "cat5.png" },
  { key: "6", parent: "2", name: "Munkustrap", source: "cat6.png" }
];
myDiagram.model = model;
</pre>

现在你已经熟悉了一些GoJS的基础知识，认真参考这些示例(https://gojs.net/latest/samples/index.html)，查看与GoJS有关的一些图表，或者阅读文档（https://gojs.net/latest/intro/index.html），可以深入了解GoJS的组件。


