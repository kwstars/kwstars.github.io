---
title: "使用Viper访问嵌套配置"
date: 2021-03-28T10:56:06+08:00
draft: false
tags: ["Viper"]
categories: ["Go"]
---



本文翻译自[《Accessing Nested Config with Viper》](https://lornajane.net/posts/2020/accessing-nested-config-with-viper)。全文如下：



我在写一个Go程序，它把其他一些东西粘在一起，所以它有一大堆基本上不相干的配置，被倾倒在一个yaml文件中。我对Viper的这种非标准使用有些纠结，但实际上，它能满足我的一切需求。而且，估计还有更多。我想我应该在这里放一些例子来说明如何处理这个问题。



## A Slice of Structs

让我们从配置的颜色开始，将其发送到我的LED架子上（我意识到我没有写博客，但我应该写！）在`config.yaml`文件中，它是一系列的对象，属性为红色、绿色和蓝色。

```yaml
shelf_lights:
  - red: 120
    blue: 200
  - red: 120
    green: 150
  - blue: 200
    green: 40
    red: 60
```



在我的代码中，我有一个叫做`LEDColour`的结构来表示这些对象中的每一个，它使用字段标签来解释yaml数据如何适合这个结构--如果你见过JSON字段标签的话，这与JSON的工作方式类似。Viper使用一个名为[mapstructure](https://github.com/mitchellh/mapstructure)的整洁包来处理这一部分。我的结构看起来像这样。

```go
type LEDColour struct {
	Red   uint8 `mapstructure:"red"`
	Green uint8 `mapstructure:"green"`
	Blue  uint8 `mapstructure:"blue"`
}
```



为了将yaml数据处理成这种结构，我声明了它将进入的变量，然后Viper有一个UnmarshalKey，允许我只从config中抓取shelf_lights部分，并对其进行处理。

```go
	var lights []LEDColour
	viper.UnmarshalKey("shelf_lights", &lights)
```



## Map of Structs with String Keys

这与上面的情况很相似，但我认为一个具体的例子可能会有帮助。在yaml配置文件中，有这样一个部分。

```yaml
obs_scenes:
  Camera:
    name: Camera 
    image: "/camera.png"
  Offline:
    name: Offline
    image: "/offline.png"
  Secrets:
    name: Secrets
    image: "/secrets.png"
```



我还有另一个结构，也具有mapstructure功能：

```go
type ObsScene struct {
	Name     string `mapstructure:"name"`
	Image    string `mapstructure:"image"`
	ButtonId int
}
```



ButtonId字段不会被更新，直到场景被动态地分配给一个按钮（这是来自我的streamdeck工具，OBS是一个视频流媒体工具，不要担心这些话没有任何意义！）。一个OBS有很多场景）所以我们在从配置中读取时不需要包括它。这看起来像这样。

```go
var buttons_obs map[string]*ObsScene
buttons_obs = make(map[string]*ObsScene)
viper.UnmarshalKey("obs_scenes", &buttons_obs)
```



像往常一样，Viper为我们提供了保障，虽然对于许多应用来说，读取整个配置文件并在需要时参考每个设置是有意义的，但这种转换配置部分的能力确实非常方便
让我们从配置的颜色开始，将其发送到我的LED架子上（我意识到我没有写博客，但我应该写！）在config.yaml文件中，它是一系列的对象，属性为红色、绿色和蓝色。