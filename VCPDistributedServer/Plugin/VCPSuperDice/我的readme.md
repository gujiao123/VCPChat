# Dice-Box · [![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/frankieali/infima-extras/blob/main/LICENSE) [![npm version](https://img.shields.io/npm/v/@3d-dice/dice-box.svg?style=flat)](https://www.npmjs.com/package/@3d-dice/dice-box)

高性能 3D 掷骰器模块，使用 [BabylonJS](https://www.babylonjs.com/)、[AmmoJS](https://github.com/kripken/ammo.js/)，并通过 [web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) 和 [offscreenCanvas](https://doc.babylonjs.com/divingDeeper/scene/offscreenCanvas) 实现。旨在便于集成到你的 JavaScript 应用中。

![演示截图](https://github.com/3d-dice/dice-box/blob/main/dice-screenshot.jpg)

## 文档

文档网站： [fantasticdice.games](https://fantasticdice.games)

## 演示
__1.0.11 版本新演示__ <br>
最新代码在 AWS 上运行： [Dice Box](https://d3rivgcgaqw1jo.cloudfront.net/index.html)

__1.0.8 版本新演示__ <br>
可以将此模块作为 ES6 模块从 [UNPKG CDN](https://unpkg.com/browse/@3d-dice/dice-box@1.0.8/) 使用<br>
无需通过 npm 安装或自行托管这些文件。 [静态 CDN 示例](https://codesandbox.io/s/dice-es6-module-cdn-lhbs99?file=/src/index.js) <br><br>
__1.0.5 版本的新演示__ <br>
在此试用完整演示（kitchen sink）： https://d3rivgcgaqw1jo.cloudfront.net/index.html <br>
查看完整演示代码： https://codesandbox.io/s/3d-dice-demo-v1-0-5-sm4ien <br>
这是一个简单的 React 属性掷骰（3d6）示例： https://codesandbox.io/s/react-roller-attributes-v1-0-5-65uqhv <br>
这是支持高级掷骰标记法的 React 示例： https://codesandbox.io/s/react-roller-advanced-notation-v1-0-5-rz0nmr

注意：某些演示包含其他 `@3d-dice` 模块，例如 [dice-roller-parser](https://github.com/3d-dice/dice-roller-parser)、[dice-ui](https://github.com/3d-dice/dice-ui) 和 [dice-parser-interface](https://github.com/3d-dice/dice-parser-interface)。高级掷骰标记（例如 `4d6dl1` 或 `4d6!r<2`）可通过这些模块支持。

## 快速开始（大致）

使用以下命令安装库：

```
npm install @3d-dice/dice-box
```

安装时，终端会询问你静态资源的目标位置。默认是 `/public/assets`，并在 10 秒后超时。你也可以手动移动这些文件。资源位于 `@3d-dice/dice-box/src/assets` 文件夹下。将该文件夹中的所有内容复制到你的本地静态资源或 public 目录。

这是一个作为构建系统一部分的 ES 模块。要在项目中导入模块，使用：

```javascript
import DiceBox from "@3d-dice/dice-box";
```

然后创建 `DiceBox` 类的新实例。参数为目标 DOM 节点的选择器，后面跟一个配置对象。请确保设置 `assetPath`（指向之前复制的资源文件夹），这是唯一的必需配置项。

```javascript
const diceBox = new DiceBox("#dice-box", {
  assetPath: "/assets/dice-box", // 必需
});
```

接着初始化类对象，初始化方法 `init` 是异步的，可以使用 `await` 或 `.then()`。

```javascript
diceBox.init().then(() => {
  diceBox.roll("2d20");
});
```

## 使用说明

Dice-Box 只能接受简单的掷骰标记与修饰符，例如 `2d20` 或 `2d6+4`。骰子停止滚动后会返回结果对象。若需更高级的掷骰功能，请考虑加入 [dice-parser-interface](https://github.com/3d-dice/dice-parser-interface)，它支持完整的 [Roll20 Dice Specification](https://help.roll20.net/hc/en-us/articles/360037773133-Dice-Reference#DiceReference-RollTemplates)。

### 配置选项
详见文档站点上的 [Configuration Options](https://fantasticdice.games/docs/usage/config#configuration-options)

### 常用对象
详见文档站点上的 [Common Objects](https://fantasticdice.games/docs/usage/objects)

### 方法
详见文档站点上的 [Methods](https://fantasticdice.games/docs/usage/methods)

### 回调
详见文档站点上的 [Callbacks](https://fantasticdice.games/docs/usage/callbacks)

## 其他设置示例

在我的演示工程中，配置如下。你可能不需要 `BoxControls`，但它们挺有趣的。Code Sandbox 演示地址： https://codesandbox.io/s/3d-dice-demo-v1-0-2-sm4ien

```javascript
import './style.css'
import DiceBox from '@3d-dice/dice-box'
import { DisplayResults, AdvancedRoller, BoxControls } from '@3d-dice/dice-ui'

let Box = new DiceBox("#dice-box",{
  assetPath: '/assets/dice-box/',
})

document.addEventListener("DOMContentLoaded", async() => {

  Box.init().then(() => {

    // 创建 dat.gui 控件
    const Controls = new BoxControls({
      themes: ["default", "rust", "diceOfRolling", "gemstone"],
      themeColor: world.config.themeColor,
      onUpdate: (updates) => {
        Box.updateConfig(updates);
      }
    });
    Controls.themeSelect.setValue(world.config.theme);
    Box.onThemeConfigLoaded = (themeData) => {
      if (themeData.themeColor) {
        Controls.themeColorPicker.setValue(themeData.themeColor);
      }
    };

    // 创建显示覆盖层
    const Display = new DisplayResults("#dice-box")

    // 创建 Roller 输入
    const Roller = new AdvancedRoller({
      target: '#dice-box',
      onSubmit: (notation) => Box.roll(notation),
      onClear: () => {
        Box.clear()
        Display.clear()
      },
      onReroll: (rolls) => {
        // 遍历解析后的掷骰标记并发送到 Box
        rolls.forEach(roll => Box.add(roll))
      },
      onResults: (results) => {
        Display.showResults(results)
      }
    })

    // 将掷骰结果传给 Advanced Roller 处理
    Box.onRollComplete = (results) => {
      Roller.handleResults(results)
    }
  }) // end Box.init
}) // end DOMContentLoaded
```

```css
html,
body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  overflow: hidden;
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

#dice-box {
  position: relative;
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  background-image: url(./assets/woodgrain2.jpg);
  background-size: cover;
}

#dice-box canvas {
  width: 100%;
  height: 100%;
}
```

## 其他项目

### [Dice UI](https://github.com/3d-dice/dice-ui)
一组为 Dice Box 提供的原生（vanilla）UI 模块。

包含：

  1. Advanced Roller Input - 支持高级掷骰标记的简单输入组件
  2. Dice Picker - 可点击的骰子图标，可用于移动端
  3. Display Results - 在模态弹窗中显示掷骰结果
  4. Box Controls - 使用 [dat.gui](https://github.com/dataarts/dat.gui) 展示可配置的 dice-box 选项

### [Dice Themes](https://github.com/3d-dice/dice-themes)
一组可用于 Dice Box 的 CC0（公共领域）模型和主题。

### [Dice Roller Parser](https://github.com/3d-dice/dice-roller-parser)
一个字符串解析器，会返回包含掷骰各组成部分的对象。它支持完整的 [Roll20 Dice Specification](https://roll20.zendesk.com/hc/en-us/articles/360037773133-Dice-Reference#DiceReference-Roll20DiceSpecification)


### [Dice Parser Interface](https://github.com/3d-dice/dice-parser-interface)
提供 `Dice Roller Parser` 和 `Dice Box` 之间的简单接口。此模块会将字符串标记发送给解析器并将其转换为 Dice Box 可用的标记对象，也会将 Dice Box 的结果返回给解析器生成最终的掷骰结果。还会处理触发重掷和爆炸骰子的事件。


## 致谢与插件

### Quest Portal
特别感谢 [Quest Portal](https://www.questportal.com/) 团队对该掷骰器开发的支持与帮助。他们提供了一些可免费分发的骰子模型，并在将 `@3d-dice/dice-box` 集成到他们平台时提供了反馈和测试。注册获取早期访问： https://app.questportal.com/signup。

### Dice So Nice
如果你需要一个基于 [three.js](https://threejs.org/) 的 3D 掷骰器，可以查看 [Dice So Nice](https://gitlab.com/riccisi/foundryvtt-dice-so-nice/-/tree/master)

### Dice of Rolling
我最喜欢的主题是基于真实产品 [Dice of Rolling](https://diceofrolling.com/#dice) 的 Dice of Rolling 主题，真实产品很棒，实际使用体验也很好。

### Owlbear Rodeo
另一个很棒的平台：如果你只需要交互式虚拟地图和掷骰， [Owlbear Rodeo](https://www.owlbear.rodeo/) 提供了非常实用的工具。将角色表带上就可以使用。
