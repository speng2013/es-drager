<template>
  <component
    :is="tag"
    ref="dragRef"
    :class="[
      'es-drager',
      { disabled, dragging: isMousedown, selected, border }
    ]"
    :style="dragStyle"
    @click.stop
  >
    <slot />

    <template v-if="showResize">
      <div
        v-for="(item, index) in dotList"
        :key="index"
        class="es-drager-dot"
        :data-side="item.side"
        :style="{ ...item }"
        @mousedown="onDotMousedown(item, $event)"
        @touchstart.passive="onDotMousedown(item, $event)"
      >
        <slot name="resize" v-bind="{ side: item.side }">
          <div class="es-drager-dot-handle"></div>
        </slot>
      </div>
    </template>
    <Rotate
      v-if="showRotate"
      :angle.sync="dragData.angle"
      :element="targetRef"
      @rotate="emitFn('rotate', dragData)"
      @rotate-start="emitFn('rotate-start', dragData)"
      @rotate-end="handleRotateEnd"
    >
      <slot name="rotate" />
    </Rotate>
  </component>
</template>
<script>
import { DragerProps } from "./drag.js";
import {
  formatData,
  withUnit,
  getDotList,
  getLength,
  degToRadian,
  getNewStyle,
  centerToTL,
  calcGrid,
  setupMove,
  getXY,
  fixBoundary,
  MouseTouchEvent,
} from "./utils";
import Rotate from "./rotate";
export default {
  props: DragerProps,
  name: "pzoomDrag",
  components: {
    Rotate,
  },
  data() {
    return {
      isMousedown: false,
      selected: false,
      dragData: {
        width: 0,
        height: 0,
        left: 0,
        top: 0,
        angle: 0,
      },
      dotList: getDotList(0, this.resizeList),
    };
  },
  watch: {
    width: {
      immediate: true,
      handler(val) {
        this.dragData.width = val;
      },
    },
    height: {
      immediate: true,
      handler(val) {
        this.dragData.height = val;
      },
    },
    top: {
      immediate: true,
      handler(val) {
        this.dragData.top = val;
      },
    },
    left: {
      immediate: true,
      handler(val) {
        this.dragData.left = val;
      },
    },
    angle: {
      immediate: true,
      handler(val) {
        this.dragData.angle = val;
      },
    },
    acitve: {
      immediate: true,
      handler(val) {
        this.selected = val;
      },
    },
    selected(val) {
      if (this.disabledKeyEvent) return;
      // 每次选中注册键盘事件
      if (val) {
        document.addEventListener("keydown", this.onKeydown);
      } else {
        // 不是选中移除
        document.removeEventListener("keydown", this.onKeydown);
      }
    },
  },
  computed: {
    dragStyle() {
      const { width, height, left, top, angle } = this.dragData;
      const style = {};
      if (width) style.width = withUnit(width);
      if (height) style.height = withUnit(height);
      return {
        ...style,
        left: withUnit(left),
        top: withUnit(top),
        zIndex: this.zIndex,
        transform: `rotate(${angle}deg)`,
        "--es-drager-color": this.color,
      };
    },
    showResize() {
      return this.resizable && !this.disabled;
    },
    showRotate() {
      return this.rotatable && !this.disabled && this.selected;
    },
    targetRef() {
      return this.$refs?.["dragRef"] || {};
    },
  },
  methods: {
    fixResizeBoundary(d, boundaryInfo, ratio) {
      const [minX, maxX, minY, maxY, parentWidth, parentHeight] = boundaryInfo;
      // 如果left小于最小x
      if (d.left < minX) {
        // 则将left赋值为最小x
        d.left = minX;
        // 宽度保持原来的不变
        d.width = this.dragData.width;
        // 如果设置等比则相应的高度也无需改变
        if (ratio) d.height = this.dragData.height;
      }

      // 如果left+width超过了父元素的宽度
      if (d.left + d.width > parentWidth) {
        // 将left赋值为老的left
        d.left = this.dragData.left;
        // 宽度变为parentWidth减去left，这样元素的left+width的和刚好等于parentWidth
        d.width = parentWidth - d.left;

        if (ratio) d.height = this.dragData.height;
      }

      // top的做法与上面类似，如果小于最小y
      if (d.top < minY) {
        // 则将top赋值为最小y
        d.top = minY;
        // 高度保持原来的不变
        d.height = this.dragData.height;

        if (ratio) d.width = this.dragData.width;
      }
      // 如果top+height超过了父元素的高度
      if (d.top + d.height > parentHeight) {
        // 将top赋值为老的top
        d.top = this.dragData.top;
        // 宽度变为parentHeight减去top，这样元素的top+height的和刚好等于parentHeight
        d.height = parentHeight - d.top;

        if (ratio) d.width = this.dragData.width;
      }
      return d;
    },
    checkDragerCollision() {
      const parentEl = this.targetRef?.parentElement || document.body;
      const broList = Array.from(parentEl.children).filter((item) => {
        return item !== this.targetRef && item.classList.contains("es-drager");
      });
      for (let i = 0; i < broList.length; i++) {
        const item = broList[i];
        const flag = checkCollision(this.targetRef, item);
        if (flag) return true;
      }
    },
    handleRotateEnd(angle) {
      this.dotList = getDotList(angle, this.resizeList);
      this.emitFn("rotate-end", this.dragData);
    },
    emitFn(type, ...args) {
      this.$emit(type, ...args);
      this.$emit("change", ...args);
    },
    onMousedown(e) {
      const mouseSet = new Set();
      console.log("onMousedown");
      console.log(e);
      mouseSet.add(e.button);
      if (this.disabled) return;
      this.isMousedown = true;
      this.selected = true;
      let { clientX: downX, clientY: downY } = getXY(e);

      const { left, top } = this.dragData;
      let minX = 0,
        maxX = 0,
        minY = 0,
        maxY = 0;
      if (this.boundary) {
        [minX, maxX, minY, maxY] = this.getBoundary();
        console.log("[minX, maxX, minY, maxY]");
        console.log([minX, maxX, minY, maxY]);
      }

      this.$emit("drag-start", this.dragData);
      const onMousemove = (e) => {
        console.log("onMousemove11");
        // 不是一个按键不执行移动
        if (mouseSet.size > 1) return;
        const { clientX, clientY } = getXY(e);
        let moveX = (clientX - downX) / this.scaleRatio + left;
        let moveY = (clientY - downY) / this.scaleRatio + top;

        // 是否开启网格对齐
        if (this.snapToGrid) {
          // 当前位置
          let { left: curX, top: curY } = this.dragData;
          // 移动距离
          const diffX = moveX - curX;
          const diffY = moveY - curY;

          // 计算网格移动距离
          moveX = curX + calcGrid(diffX, this.gridX);
          moveY = curY + calcGrid(diffY, this.gridY);
        }

        if (this.boundary) {
          [moveX, moveY] = fixBoundary(moveX, moveY, minX, maxX, minY, maxY);
          console.log("[moveX, moveY]");
          console.log([moveX, moveY]);
        }
        this.dragData.left = moveX;
        this.dragData.top = moveY;
        this.$emit("drag", this.dragData);
      };
      setupMove(onMousemove, (e) => {
        if (this.checkCollision) {
          const isCollision = this.checkDragerCollision();
          if (isCollision) {
            this.dragData.top = top;
            this.dragData.left = left;
          }
        }
        mouseSet.clear();
        this.isMousedown = false;
        const clickOutsize = () => {
          this.selected = false;
        };
        document.addEventListener("click", clickOutsize, { once: true });
        this.$emit("drag-end", this.dragData);
      });
    },
    /**
     * 缩放
     * @param dotInfo
     * @param e
     */
    onDotMousedown(dotInfo, e) {
      console.log("onDotMousedown");
      e.stopPropagation();
      // 获取鼠标按下的坐标
      const { clientX, clientY } = getXY(e);
      const downX = clientX;
      const downY = clientY;
      const { width, height, left, top } = this.dragData;

      // 中心点
      const centerX = left + width / 2;
      const centerY = top + height / 2;

      const rect = {
        width,
        height,
        centerX,
        centerY,
        rotateAngle: this.dragData.angle,
      };
      const type = dotInfo.side;

      const { minWidth, minHeight, aspectRatio, equalProportion } = this;
      this.emitFn("resize-start", this.dragData);
      let boundaryInfo = [];
      if (this.boundary) {
        boundaryInfo = this.getBoundary();
      }

      const onMousemove = (e) => {
        console.log("onMousemove");
        const { clientX, clientY } = getXY(e);
        // 距离
        let deltaX = (clientX - downX) / this.scaleRatio;
        let deltaY = (clientY - downY) / this.scaleRatio;
        // 开启网格缩放
        if (this.snapToGrid) {
          deltaX = calcGrid(deltaX, this.gridX);
          deltaY = calcGrid(deltaY, this.gridY);
        }

        const alpha = Math.atan2(deltaY, deltaX);
        const deltaL = getLength(deltaX, deltaY);
        const isShiftKey = e.shiftKey;

        const beta = alpha - degToRadian(rect.rotateAngle);
        const deltaW = deltaL * Math.cos(beta);
        const deltaH = deltaL * Math.sin(beta);
        const ratio =
          (equalProportion || isShiftKey) && !aspectRatio ? rect.width / rect.height : aspectRatio;

        const {
          position: { centerX, centerY },
          size: { width, height },
        } = getNewStyle(
          type,
          { ...rect, rotateAngle: rect.rotateAngle },
          deltaW,
          deltaH,
          ratio,
          minWidth,
          minHeight
        );

        const pData = centerToTL({
          centerX,
          centerY,
          width,
          height,
          angle: this.dragData.angle,
        });

        let d = {
          ...this.dragData,
          ...formatData(pData, centerX, centerY),
        };

        // 最大宽高限制
        if (this.maxWidth > 0) {
          d.width = Math.min(d.width, this.maxWidth);
        }
        if (this.maxHeight > 0) {
          d.height = Math.min(d.height, this.maxHeight);
        }

        // 如果开启了边界，则调用 fixResizeBoundary 函数处理
        if (this.boundary) {
          console.log("fixResizeBoundary");
          d = this.fixResizeBoundary(d, boundaryInfo, ratio);
        }

        this.dragData = d;
        this.emitFn("resize", this.dragData);
      };

      setupMove(onMousemove, () => {
        // 碰撞检测
        if (this.checkCollision && this.checkDragerCollision()) {
          // 发生碰撞回到原来位置
          this.dragData = { ...this.dragData, width, height, left, top };
        }
        this.emitFn("resize-end", this.dragData);
      });
    },
    onKeydown(e) {
      console.log("onKeydown");
      if (this.isMousedown) return;
      let { left: moveX, top: moveY } = this.dragData;
      if (["ArrowRight", "ArrowLeft"].includes(e.key)) {
        // 左右键修改left
        const isRight = e.key === "ArrowRight";
        // 默认移动1像素距离
        let diff = isRight ? 1 : -1;
        // 如果开启网格，移动gridX距离
        if (this.snapToGrid) {
          diff = isRight ? this.gridX : -this.gridX;
        }
        moveX = moveX + diff;
      } else if (["ArrowUp", "ArrowDown"].includes(e.key)) {
        // 左右键修改top
        const isDown = e.key === "ArrowDown";
        // 默认移动1像素距离
        let diff = isDown ? 1 : -1;
        // 如果开启网格，移动gridY距离
        if (this.snapToGrid) {
          diff = isDown ? this.gridY : -this.gridY;
        }

        moveY = moveY + diff;
      }

      // 边界判断
      if (this.boundary) {
        const [minX, maxX, minY, maxY] = this.getBoundary();
        [moveX, moveY] = fixBoundary(moveX, moveY, minX, maxX, minY, maxY);
      }
      // 一次只会有一个会变
      this.dragData.left = moveX;
      this.dragData.top = moveY;
    },
    getBoundary() {
      let minX = 0,
        minY = 0;
      const { left, top, height, width, angle } = this.dragData;
      const parentEl = this.targetRef?.parentElement || document.body;
      console.log("parentEl----------------------------------------");
      console.log(parentEl);
      console.log(parentEl?.getBoundingClientRect());
      const parentElRect = parentEl?.getBoundingClientRect();
      if (angle && this.scaleRatio === 1) {
        const rect = this.targetRef?.getBoundingClientRect();
        minX = Math.abs(rect.left - (left + parentElRect.left));
        minY = Math.abs(rect.top - (top + parentElRect.top));
      }

      // 最大x
      const maxX = parentElRect.width / this.scaleRatio - width;
      // 最大y
      const maxY = parentElRect.height / this.scaleRatio - height;
      console.log([minX, maxX - minX, minY, maxY - minY, parentElRect.width, parentElRect.height]);
      return [minX, maxX - minX, minY, maxY - minY, parentElRect.width, parentElRect.height];
    },
  },
  mounted() {
    if (!this.targetRef) return;
    // 没传宽高使用元素默认
    if (!this.dragData.width && !this.dragData.height) {
      const { width, height } = this.targetRef.getBoundingClientRect();
      // 获取默认宽高
      dragData = {
        ...dragData,
        width: width || 100,
        height: height || 100,
      };
      this.$emit("change", { ...this.dragData });
    }
    this.targetRef.addEventListener("mousedown", this.onMousedown);
    this.targetRef.addEventListener("touchstart", this.onMousedown, {
      passive: true,
    });
  },
  destroyed() {
    document.removeEventListener("keydown", this.onKeydown);
  },
};
</script>
<style lang="scss">
.es-drager {
  --es-drager-color: #3a7afe;
  position: absolute;
  &::after {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: none;
  }
  &.selected {
    transition: none;
    user-select: none;
    &::after {
      display: block;
      outline: 1px dashed var(--es-drager-color);
    }
    .es-drager-dot {
      display: block;
    }
  }
  &.border {
    border: 1px solid var(--es-drager-color);
  }
  &.disabled {
    opacity: 0.4;
    cursor: not-allowed !important;
  }
  &:hover {
    cursor: move;
  }
  &-dot {
    display: none;
    position: absolute;
    z-index: 1;
    transform: translate(-50%, -50%);
    cursor: se-resize;
    &[data-side*="right"] {
      transform: translate(50%, -50%);
    }
    &[data-side*="bottom"] {
      transform: translate(-50%, 50%);
    }
    &[data-side="bottom-right"] {
      transform: translate(50%, 50%);
    }

    &-handle {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background-color: var(--es-drager-color);
    }
  }
}
</style>