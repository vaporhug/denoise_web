<!-- src/components/ImageCompare.vue -->
<template>
  <div
      class="image-compare-container"
      ref="container"
      @mousemove="onMouseMove"
      @mouseleave="onMouseLeave"
      @touchmove="onTouchMove"
      @touchend="stopDrag"
  >
    <!-- 原图 -->
    <img :src="originalImage" class="image original" ref="originalImg" @load="onImageLoad" />

    <!-- 去噪后的图像，显示从滑动线位置到右侧 -->
    <div
        class="overlay"
        :style="{  width: sliderPosition + '%' }"
        ref="overlay"
    >
      <img :src="denoisedImage" class="image denoised" />
    </div>

    <!-- 滑动线及句柄 -->
    <div
        class="slider"
        :style="{ left: sliderPosition + '%' }"
        @mousedown="startDrag"
        @touchstart="startDrag"
    >
      <div class="slider-line"></div>
      <div class="slider-handle"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ImageCompare',
  props: {
    originalImage: {
      type: String,
      required: true
    },
    denoisedImage: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      dragging: false,
      sliderPosition: 50, // 初始位置 50%
      containerWidth: 0,
      containerLeft: 0
    };
  },
  mounted() {
    window.addEventListener('mouseup', this.stopDrag);
    window.addEventListener('touchend', this.stopDrag);
    window.addEventListener('resize', this.onImageLoad); // 处理窗口大小变化
  },
  beforeUnmount() {
    window.removeEventListener('mouseup', this.stopDrag);
    window.removeEventListener('touchend', this.stopDrag);
    window.removeEventListener('resize', this.onImageLoad);
  },
  methods: {
    onImageLoad() {
      this.$nextTick(() => {
        const container = this.$refs.container;
        const rect = container.getBoundingClientRect();
        this.containerWidth = rect.width;
        this.containerLeft = rect.left + window.scrollX;
      });
    },
    startDrag(event) {
      event.preventDefault();
      this.dragging = true;
      this.updateSlider(event);
      // 添加事件监听
      window.addEventListener('mousemove', this.handleMouseMove);
      window.addEventListener('touchmove', this.handleTouchMove, { passive: false });
    },
    stopDrag() {
      if (this.dragging) {
        this.dragging = false;
        window.removeEventListener('mousemove', this.handleMouseMove);
        window.removeEventListener('touchmove', this.handleTouchMove);
      }
    },
    handleMouseMove(event) {
      if (this.dragging) {
        this.updateSlider(event);
      }
    },
    handleTouchMove(event) {
      if (this.dragging) {
        this.updateSlider(event.touches[0]);
        event.preventDefault(); // 防止页面滚动
      }
    },
    updateSlider(event) {
      let clientX = event.clientX;

      // 计算相对于容器的 x 坐标
      let x = clientX - this.containerLeft;
      if (x < 0) x = 0;
      if (x > this.containerWidth) x = this.containerWidth;
      this.sliderPosition = (x / this.containerWidth) * 100;
    },
    onMouseMove(event) {
      if (this.dragging) {
        this.updateSlider(event);
      }
    },
    onTouchMove(event) {
      if (this.dragging) {
        this.updateSlider(event.touches[0]);
        event.preventDefault(); // 防止页面滚动
      }
    },
    onMouseLeave() {
      this.dragging = false;
      this.stopDrag();
    }
  }
};
</script>

<style scoped>
.image-compare-container {
  position: relative;
  display: inline-block;
  max-width: 800px;
  width: 100%;
  user-select: none;
  touch-action: none;
}

.image {
  display: block;
  width: 100%;
  height: auto;
}

.overlay {
  position: absolute;
  top: 0;
  /* left 和 width 通过绑定的样式控制 */
  height: 100%;
  overflow: hidden;
}

.denoised {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover; /* 确保图像覆盖整个容器 */
}

.slider {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 2px;
  background: #fff;
  cursor: ew-resize;
  transition: background 0.3s;
  z-index: 10;
}

.slider-line {
  position: absolute;
  top: 0;
  bottom: 0;
  left: -1px;
  width: 2px;
  background: rgba(255, 255, 255, 0.7);
}

.slider-handle {
  position: absolute;
  top: 50%;
  left: -10px;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  background: #fff;
  border: 2px solid #000;
  border-radius: 50%;
}

.overlay {
  position: absolute;
  top: 0;
  height: 100%;
  overflow: hidden; /* 确保右边溢出的部分被裁剪 */
}

.denoised {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover; /* 确保图像以填充容器并保持比例，超出部分会被裁剪 */
  object-position: left center; /* 使图像的左端与容器的左端对齐 */
}


</style>
