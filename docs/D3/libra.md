# Libra.js

[官网地址](https://libra-js.github.io/)

## 快速开始

### 第一步：确定图层

准备由 D3 编写的可视化图表，根据不同交互将其组织为不同的层：

- 主层：根据可视化表达渲染数据集；
- 背景层：在主层下，在恰当的时刻显示坐标轴等元素；
- 选择层：在主层上，展示被选中的元素；
- 临时层：在选择层上，展示选择区域。

层可以通过 `Libra.Layer.initialize` 进行管理。

```javascript
const mainLayer = Libra.Layer.initialize(
    "D3Layer", 
    {
        name: "mainLayer",
        width: WIDTH,
        height: HEIGHT,
        container: svg.node(),
    }
);
```

### 第二步：自定义组件操作

如果有需要，或者预设组件无法达到需要，则可以进行自定义。

```js
// custom by registering when the built-in ones do not meet your requirements
Libra.GraphicalTransformer.register(
    "DrawTransient",
    {
        redraw: ({ layer }) => {...}, // draw the transient feedback, e.g. a selection delegate
        sharedVar: {}
    }
);

// initialize the built-in transformer or the custom one defined before
Libra.GraphicalTransformer.initialize(
    "DrawTransient", // the name of the transformer
    {
        layer: transientLayer,
        sharedVar: {}
    }
)

Libra.Service.register(
    "CustomService", // the name of the service
    { ... }
)

Instrument.register(
    "ExcentricLabelingInstrument", // the name of the instrument
    { ... }
)
```

### 第三步：工具初始化

初始化工具，并将其绑定到指定层。

```js
const instrument = Libra.Instrument.initialize(
    "HoverInstrument", // the name of the built-in or custom instrument defined in step 2
    {
        layers: [{ layer: mainLayer }], // bind the layer
        interactors: [ ... ], // *extend the interactor to support more events
        sharedVar: {
            highlightAttrValues: {
                fill: "blue"
            }
        }
    }
);
```

### 第四步：组件绑定

将服务与转换绑定到需要的工具上。

```js
instrument.services
    .find("SomeService") // some built-in services or custom service defined in step 2
    .transformers.add("SomeTransformer"); // the transformer defined in step 2
```

### 完整代码

```js
/** Step 1: rendering the visualization and organize it into layers */
const svg = d3
	.select("#LibraPlayground")
	.append("svg")
	.attr("width", 500)
	.attr("height", 500);

const data = new Array(100)
	.fill()
	.map(() => ({
		x: Math.random() * 480 + 10,
		y: Math.random() * 480 + 10
	}));

const mainLayer = Libra.Layer.initialize("D3Layer", {
	name: "mainLayer",
	width: 500,
	height: 500,
	container: svg.node(),
});

const g = d3.select(mainLayer.getGraphic());
g.selectAll("circle")
	.data(data)
	.enter()
	.append("circle")
	.attr("cx", (d) => d.x)
	.attr("cy", (d) => d.y)
	.attr("r", 10)
	.attr("fill", "red")
	.attr("stroke", "black");

/** Step 2: initialize transformer, custom service and instrument if necessary*/
// Do something

/** Step 3: initialize instrument and mount it to corresponding layer */
const instrument = Libra.Instrument.initialize("HoverInstrument", {
	layers: [mainLayer],
	sharedVar: {
		highlightAttrValues: {
			fill: "blue"
		}
	}
});

/** Step 4: bind services and transformers if necessary*/
instrument.services
	.find("SomeService")
	.transformers.add("SomeTransformer");
```

## 工具 Instrument

### 初始化

通过 `Libra.Instrument.initialize(className, option)` 进行初始化，参数说明如下：

|参数|类型|描述|例子|
|:-:|:-:|:-:|:-:|
|className|string|工具类名|"HoverInstrument"|
|option|InstrumentOption|设置选项| |

InstrumentOption 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| name | string | 工具名，方便进行引用 |  |
| on | {[action: string]: (FeedForward \| Command)[]} | 高层动作的回调 |  |
| services | (string \| Service \| {service: Service, options: any})[] | 关联服务（某些内置工具具有默认服务 | [kMeansService] |
| transformers | GraphicalTransformer[] | 关联变换（某些内置工具具有默认变换） | [transientTransformer, mainTransformer] |
| layers | Layer[] \| {layer: Layer, options: any}[] | 涉及的层 | [mainLayer] |
| sharedVar | {[varName: string]: any} | 工具共享的变量 |  |

### 注册

通过 `Libra.Instrument.register(name, definition)` 进行注册，参数说明如下：

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 工具类名，将会在初始化步骤使用 | "ExcentricLabelingInstrument" |
| definition | InstrumentDefinition |  |  |

InstrumentDefinition 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| constructor | Instrument | 该类的基类，可以在此基础上扩展 |  |
| …InstrumentOption | …InstrumentOption |  |  |

### 注销

通过 `Libra.Instrument.unregister(name)` 进行注销，`name` 与注册时 `name` 相同：

### 查找

通过 `Instrument.findInstrument(classNameOrName)` 查找工具实例，可以是工具名，或者是工具类名。

### 内置工具

| 工具名 | 描述 |
| :-: | :-: |
| HoverInstrument | 通过鼠标覆盖(hover)选中图形元素，并改变其属性 |
| HelperLineInstrument | 在鼠标点下显示一条线，同时突出选中元素 |
| ClickInstrument | 通过鼠标点击选中图形元素，并改变其属性 |
| BrushInstrument | 通过鼠标按住滑动选取矩形区域，选取其内部图形元素，并改变其属性 |
| BrushXInstrument | 通过鼠标按住滑动选取固定高度的矩形区域，选取其内部图形元素，并改变其属性 |
| BrushYInstrument | 通过鼠标按住滑动选取固定宽度的矩形区域，选取其内部图形元素，并改变其属性 |
| DragInstrument | 改变图形元素的位置属性 |
| PanInstrument | 拖拽图层 |
| GeometricZoomInstrument | 缩放图层 |
| SemanticZoomInstrument | 缩放图层，并保留范围信息 |

### 实例 API

实例由 `Instrument.initialize` 创造。

#### on

为动作添加前向反馈或指令。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| action | string \| string[] | 高层动作 |
| feedForwardOrCommand | Function | 在触发对应动作时，执行前向反馈或命令 |

#### off

删除动作的前向反馈或指令。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| action | string \| string[] | 高层动作 |
| feedForwardOrCommand | Function | 取消执行前向反馈或命令 |

#### attach

将工具连接到指定层。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| layer | Layer | 需要连接的层 |
| options | {pointerEvents: "all" \| "viewport" \| "visiblePainted"} | 设置事件或动作的可接受范围<br>all 允许任何位置<br>viewport 会根据层的设置进行限定<br>visiblePainted 只允许绘图区域 |

#### getSharedVar

获取工具的共享变量，返回对应的变量值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| sharedName | string | 共享变量的名字 |

#### setSharedVar

设置共享变量的值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| sharedName | string | 共享变量的名字 |
| value | any | 共享变量的值 |

#### isInstanceOf

判定该工具实例是否是某工具类的实例。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| className | string | 工具类名字 |

## 交互器

### 初始化

通过 `Libra.Interactor.initialize(className, option)` 进行初始化，参数说明如下：

| 参数 |类型|描述|例子|
|:-:|:-:|:-:|:-:|
| className | string | 交互器类名 | "MouseTraceInteractor" |
| option | InteractorOption | 设置选项 |  |

InteractorOption 对象包含以下属性：

| 属性 | 类型 | 描述 |
| :-: | :-: | :-: |
| actions | { action: string, events: string[], transitions?: [string, string][] }[] | 继承自不同的基类具有不同的效果<br>action: 高层动作<br>events: 允许的底层事件<br>transitions: 两个状态间的改变 |

### 注册

通过 `Libra.Interactor.register(className, definition)` 进行注册，参数说明如下：

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 交互器类名，将会在初始化步骤使用 | "MultiModalInteractor" |
| definition | InteractorDefinition |  |  |

InteractorDefinition 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| constructor | Interactor | 该类的基类，可以在此基础上扩展 |  |
| state | string | 自动机初始状态 | "start" |
| actions | { action: string, events: string[], transitions?: [string, string][] }[] | 继承自不同的基类具有不同的效果<br>action: 高层动作<br>events: 允许的底层事件<br>transitions: 两个状态间的改变 |  |

### 注销

通过 `Libra.Interactor.unregister(className)` 进行注销，`className` 与注册时 `className` 相同。

### 内置交互器

| 交互器名 | 描述 |
| :-: | :-: |
| MousePositionInteractor | 鼠标移动 |
| TouchPositionInteractor | 开始触碰 $\to$ 结束触碰 |
| MouseTraceInteractor | 鼠标按下 $\to$ 鼠标移动 $\to$ 右键点击 |
| TouchTraceInteractor | 开始触碰 $\to$ 接触移动 $\to$ 结束触碰 |
| KeyboardPositionInteractor | 按下按键(空格) $\to$ 按下按键(方向键[WSAD$\uparrow\downarrow\leftarrow\rightarrow$]) |
| MouseWheelInteractor | 鼠标第一次移动到激活区域 $\to$ 鼠标移出元素 $\to$ 右键点击 |

### 实例 API

#### getActions

返回交互器实例中的动作。

#### setActions

为交互器实例添加新的动作。

#### getInputEventTypes

返回允许的输入事件类型。

#### dispatch

派发任务给交互器。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| event | string \| Event | 待派发任务 |
| layer | Layer? | 产生影响的层 |

#### isInstanceOf

判定该交互器实例是否是某交互器类的实例。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| className | string | 交互器类名字 |

## 图层

### 初始化

通过 `Libra.Layer.initialize(className, option)` 进行初始化，参数说明如下：

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 渲染图层的类别 | "D3Layer" |
| option | LayerOption | 设置选项 |  |

LayerOption 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| name | string | 图层名称 | "mainLayer" |
| width | number | 图层宽度 | 500 |
| height | number | 图层高度 | 500 |
| offset | {x: number, y: number} | 容器坐标偏差值 |  |
| container | D3 Group | 容器坐标偏差值 | 带有 "g" 元素的 D3 选择 |

### 注册

如果使用 D3 以外的可视化引擎，则需要注册，通过 `Libra.Layer.register(className, definition)` 进行，参数说明如下：

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 自定义图层的类别 | "VegaLayer" |
| definition | LayerDefinition |  |  |

LayerDefinition 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| constructor | Layer | 该类的基类，可以在此基础上扩展 |  |
| ...LayerOption | ...LayerOption |  |

### 注销

通过 `Libra.Layer.unregister(className)` 进行注销，`className` 与注册时 `className` 相同。

### 查找

通过 `Instrument.findInstrument(classNameOrName)` 查找图层实例，可以是图层名，或者是图层类名。

### 实例 API

#### getGraphic

获取图层的 DOM 元素。

#### getContainerGraphic

获取图层容器的 DOM 元素。

#### getVisualElement

获取图层上的可见元素。

#### getLayerFromQueue

根据名称，获取同级图层。

#### setLayersOrder

调整图层的栈结构。当值为 -1 时表示该图层不可见。

| 参数 | 类型 | 描述 | 
| :-: | :-: | :-: |
| layerNameOrderKVPairs | {[key: string]: number} | 定义了图层的顺序，每个图层具有对应权值，权值越高越会显示在顶部 |

#### picking

获取图层上的可视元素。

#### isInstanceOf

判定图层实例是否是指定的图层类型（图层类）。

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 图层类(图层类) | "D3Layer" | 

## 转换器

### 初始化

通过 `Libra.GraphicalTransformer.initialize(className, option)` 进行初始化，参数说明如下：

| 参数 |类型|描述|例子|
|:-:|:-:|:-:|:-:|
| className | string | 转换器类名 | "drawDelegateTransformer" |
| option | TransformerOption | 设置选项 |  |

TransformerOption 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| layer | Layer | 转换器作用图层 | mainLayer |
| sharedVar | object | 服务与转换器间的共享变量 |  |
| redraw | Function ({layer: Layer, transformer: Transformer}) => void | 渲染函数，用于渲染可视化 |  |
| transient | boolean | 渲染内容是临时的还是永久的 | true |

### 注册

通过 `Libra.GraphicalTransformer.register(className, definition)` 进行注册，参数说明如下：

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 转换器类名，将会在初始化步骤使用 | "drawDelegateTransformer" |
| option | TransformerOption |  |  |

### 注销

通过 `Libra.GraphicalTransformer.unregister(className)` 进行注销，`className` 与注册时 `className` 相同。

### 内置转换器

| 转换器名 | 描述 |
| :-: | :-: |
| TransientRectangleTransformer | 绘制临时矩形，在 BrushInstrument、BrushXInstrument、BrushYInstrument 中使用 |
| SelectionTransformer | 绘制图层上的选取元素，在服务中的选择服务中被使用 |
| HelperLineTransformer | 在屏幕上绘制辅助线，支持水平或竖直，在 HelperLineInstrument 中使用 |

### 实例 API

#### getSharedVar

获取工具的共享变量，返回对应的变量值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| sharedName | string | 共享变量的名字 |

#### setSharedVar

设置共享变量的值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| sharedName | string | 共享变量的名字 |
| value | any | 共享变量的值 |

#### setSharedVars

设置多个共享变量的值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| object | {[key: string]: any} | 以键-值对设置共享变量 |

## 服务

### 初始化

通过 `Libra.Service.initialize(className)` 进行初始化。

### 注册

通过 `Libra.Service.register(className, definition)` 进行注册，参数说明如下：

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 服务类名，将会在初始化步骤使用 | "filterPointService" |
| option | ServiceDefinition |  |  |

ServiceDefinition 对象包含以下属性：

| 属性 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| constructor | Service | 该类的基类，可以在此基础上扩展 | LayoutService \| SelectionService \| … other defined Service |
| params | {[key: string]: any} | 评估函数初始参数 |  |
| sharedVar | {[key: string]: any} | 共享变量。当给出共享变量后，将会执行初始化过的评估函数 |  |
| evaluate | Function (sharedVarsAndParams) => any | 评估函数，参数为输入参数和用户定义的共享变量，返回值将传给相关联的转换器 |  |

### 注销

通过 `Libra.Service.unregister(className)` 进行注销，`className` 与注册时 `className` 相同。

### 内置转换器

| 转换器名 | 描述 |
| :-: | :-: |
| SelectionService | 从图层选择元素的服务 |
| LayoutService | 计算元素的新的布局 |
| AnalysisService | 分析及运算 |

### 实例 API

#### getSharedVar

获取工具的共享变量，返回对应的变量值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| sharedName | string | 共享变量的名字 |

#### setSharedVar

设置共享变量的值。

| 参数 | 类型 | 描述 |
| :-: | :-: | :-: |
| sharedName | string | 共享变量的名字 |
| value | any | 共享变量的值 |

#### isInstanceOf

判定服务实例是否是指定的服务类型。

| 参数 | 类型 | 描述 | 例子 |
| :-: | :-: | :-: | :-: |
| className | string | 服务类型 | "SelectionService" | 