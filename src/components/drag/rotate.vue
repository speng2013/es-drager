<template>
  <div
    ref="rotateRef"
    class="es-drager-rotate"
    @mousedown="onRotateMousedown"
    @touchstart.passive="onRotateMousedown"
  >
    <slot>
      <div class="es-drager-rotate-handle">
        <svg viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg">
          <path
            fill="currentColor"
            d="M784.512 230.272v-50.56a32 32 0 1 1 64 0v149.056a32 32 0 0 1-32 32H667.52a32 32 0 1 1 0-64h92.992A320 320 0 1 0 524.8 833.152a320 320 0 0 0 320-320h64a384 384 0 0 1-384 384 384 384 0 0 1-384-384 384 384 0 0 1 643.712-282.88z"
          />
        </svg>
      </div>
    </slot>
  </div>
</template>

<script>
import { getXY, MouseTouchEvent, setupMove } from "./utils";
export default {
  props: {
    angle: {
      type: Number,
      default: 0,
    },
    element: {
      type: Element,
    },
  },
  data() {
    return {};
  },
  methods: {
    onRotateMousedown(e) {
      if (!this.element)
        return console.warn("[es-drager] rotate component need drag element property");
      e.stopPropagation();
      const { width, height, left, top } = this.element.getBoundingClientRect();
      // 旋转中心位置
      const centerX = left + width / 2;
      const centerY = top + height / 2;

      this.$emit("rotate-start", this.value);
      setupMove(
        (e) => {
          const { clientX, clientY } = getXY(e);
          const diffX = centerX - clientX;
          const diffY = centerY - clientY;
          // Math.atan2(y,x) 返回x轴到(x,y)的角度 // pi值
          const radians = Math.atan2(diffY, diffX);
          const deg = (radians * 180) / Math.PI - 90;
          console.log("deg");
          console.log(deg);
          //   this.angle = (deg + 360) % 360;
          this.$emit("update:angle", (deg + 360) % 360);
        },
        () => {
          this.$emit("rotate-end", this.angle);
        }
      );
    },
  },
};
</script>
<style lang="scss">
.es-drager-rotate {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translate(-50%, -200%);
  &-handle {
    width: 16px;
    height: 16px;
    font-size: 20px;
    color: var(--es-drager-color);
  }
}
</style>