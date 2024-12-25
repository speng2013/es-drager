# 基于https://github.com/vangleer/es-drager 改造而来适用于vue2.x，感谢原作者;
# 安装依赖
npm install

# 运行项目
npm run serve 

## Drager API

### Drager 属性

| 属性名                   | 说明           | 类型                                         | 默认    |
| --------------------- | ------------ | ------------------------------------------ | ----- |
| tag | component组件的is属性       | ^[string]         | div     |
| type | 类型，`rect`, `text`, `image`       | ^[string]         | rect     |
| width | 宽度       | ^[number]         | 100     |
| height | 高度       | ^[number]         | 100     |
| left | 横坐标偏移       | ^[number]         | 0     |
| top | 纵坐标偏移       | ^[number]         | 0     |
| angle | 旋转角度       | ^[number]         | 0     |
| color | 颜色       | ^[string]         |   #3a7afe   |
| resizable | 是否可缩放       | ^[boolean]        | true     |
| rotatable | 是否可旋转       | ^[boolean]        | -     |
| boundary | 是否判断边界(最近定位父节点，考虑性能谨慎使用)     | ^[boolean]        | -     |
| disabled | 是否禁用     | ^[boolean]        | -     |
| minWidth | 最小宽度     | ^[number]        | -     |
| minHeight | 最小高度     | ^[number]        | -     |
| maxWidth | 最大宽度     | ^[number]        | -     |
| maxHeight | 最大高度     | ^[number]        | -     |
| selected | 控制是否选中     | ^[boolean]        | -     |
| checkCollision | 是否开启碰撞检测     | ^[boolean]        | -     |
| snapToGrid | 开启网格     | ^[boolean]        | -     |
| gridX | 网格X大小     | ^[number]        | 50     |
| gridY | 网格Y大小     | ^[number]        | 50     |
| snap | 开启吸附     | ^[boolean]        | -     |
| snapThreshold | 吸附阈值     | ^[number]        | 10     |
| markline | 辅助线([可自定义](https://vangleer.github.io/es-drager/#/markline))     | ^[boolean]^[Function]       | -     |
| extraLines | 添加除了es-drager元素以外的对齐线，例如添加中心点对齐([可参考](https://vangleer.github.io/es-drager/#/markline))     | ^[Function]  |      |
| scaleRatio | 缩放比     | ^[number]        | 1     |
| disabledKeyEvent | 禁用方向键移动     | ^[boolean]        | -     |
| border | 是否显示边框     | ^[boolean]        | true     |
| aspectRatio | 宽高缩放比     | ^[number]        | -     |
| equalProportion | 宽高等比缩放(该属性和aspectRatio互斥，同时使用会存在问题)     | ^[boolean]        | -     |
| resizeList |  显示的缩放handle列表，`top`, `bottom`, `left`, `right`, `top-left`, `top-right`, `bottom-left`, `bottom-right`   | ^[string[]]        | -  |



### Drager 事件

| 事件名    | 说明          | 类型                                                             |
| ------ | ----------- | -------------------------------------------------------------- |
| change | 位置、大小改变 | ^[Function]`(dragData) => void` |
| drag | 拖拽中 | ^[Function]`(dragData) => void` |
| drag-start | 拖拽开始 | ^[Function]`(dragData) => void` |
| drag-end | 拖拽结束 | ^[Function]`(dragData) => void` |
| resize | 缩放中 | ^[Function]`(dragData) => void` |
| resize-start | 缩放开始 | ^[Function]`(dragData) => void` |
| resize-end | 缩放结束 | ^[Function]`(dragData) => void` |
| rotate | 旋转中 | ^[Function]`(dragData) => void` |
| rotate-start | 旋转开始 | ^[Function]`(dragData) => void` |
| rotate-end | 旋转结束 | ^[Function]`(dragData) => void` |
| focus | 获取焦点/选中 | ^[Function]`(selected) => void` |
| blur | 失去焦点/非选中 | ^[Function]`(selected) => void` |

- dragData 类型

```javascript
export type DragData = {
  width: number
  height: number
  left: number
  top: number
  angle: number
}
```


### Drager 插槽

| 插槽名     | 说明      |
| ------- | ------- |
| default | 自定义默认内容 |
| resize | 缩放handle |
| rotate | 旋转handle |

具体功能可以到原作者那里查看，功能是一致的。
