<template>
  <div class="vc-display">
    <div class="vc-current-color vc-transparent">
      <div class="color-cube" :style="getBgColorStyle"></div>
    </div>
    <div class="vc-color-input">
      <input :value="state.hex" @input="onInputChange" />
    </div>
    <div class="vc-alpha-input" v-if="!disableAlpha">
      <input class="vc-alpha-input__inner" :value="state.alpha" @input="onAlphaBlur" />
    </div>
  </div>
</template>

<script lang="ts">
  import { computed, defineComponent, reactive } from "vue";
  import propTypes from "vue-types";
  import { Color } from "../utils/color";
  import { whenever, useDebounceFn } from "@vueuse/core";
  import tinycolor from "tinycolor2";

  export default defineComponent({
    name: "Display",
    props: {
      color: propTypes.instanceOf(Color),
      disableAlpha: propTypes.bool.def(false),
    },
    emits: ["update:color", "change"],
    setup(props, { emit }) {
      const state = reactive({
        color: props.color,
        hex: props.color?.hex,
        alpha: props.color?.alpha + "%",
        previewBgColor: props.color?.toRgbString(),
      });

      const getBgColorStyle = computed(() => {
        return {
          background: state.previewBgColor,
        };
      });

      const onAlphaBlur = useDebounceFn((event) => {
        if (!event.target.value) {
          return;
        }

        const opacity = parseInt(event.target.value.replace("%", ""));
        if (!isNaN(opacity) && state.color) {
          state.color.alpha = opacity;
        }

        emit("update:color", state.color);
        emit("change", state.color);
      }, 300);

      const onInputChange = useDebounceFn((event) => {
        if (!event.target.value) {
          return;
        }

        const _hex = event.target.value.replace("#", "");
        if (tinycolor(_hex).isValid() && state.color) {
          state.color.hex = _hex;
        }

        emit("update:color", state.color);
        emit("change", state.color);
      }, 300);

      whenever(
        () => props.color,
        (value: Color) => {
          if (value) {
            state.color = value;
            state.alpha = state.color.alpha + "%";
            state.hex = state.color.hex;
          }
        },
        { deep: true }
      );

      whenever(
        () => state.color,
        () => {
          if (state.color) {
            state.previewBgColor = state.color.toRgbString();
          }
        },
        { deep: true }
      );

      return { state, getBgColorStyle, onAlphaBlur, onInputChange };
    },
  });
</script>

<style lang="scss" scoped>
  .vc-display {
    margin-top: 16px;
    height: 28px;
    display: flex;
    align-items: center;

    .vc-current-color {
      margin-right: 10px;
      width: 50px;
      height: 100%;
      box-shadow: 3px 0 5px #00000014;
      position: relative;
      cursor: pointer;
      overflow: hidden;
      display: inline-block;
      vertical-align: middle;

      &.vc-transparent {
        background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAMCAIAAADZF8uwAAAAGUlEQVQYV2M4gwH+YwCGIasIUwhT25BVBADtzYNYrHvv4gAAAABJRU5ErkJggg==);
        background-repeat: repeat;
      }

      .color-cube {
        width: 100%;
        height: 100%;
      }
    }

    .vc-color-input {
      height: 100%;
      flex: 1;
      flex-shrink: 0;
      box-sizing: border-box;
      position: relative;

      input {
        padding: 0;
        border: 0;
        outline: none;
        cursor: pointer;
        color: #33383e;
        font-size: 14px;
        text-align: center;
        box-sizing: border-box;
        background-color: #f1f2f4;
        border-radius: 4px;
        height: 100%;
        width: 100%;
      }
    }

    .vc-alpha-input {
      margin-left: 8px;
      width: 56px;
      height: 100%;
      border: none;
      display: -ms-flexbox;
      display: flex;
      -ms-flex-pack: center;
      justify-content: center;
      -ms-flex-align: center;
      align-items: center;
      background-color: #f1f2f4;
      border-radius: 4px;
      font-size: 14px;

      > input {
        width: 100%;
        padding: 0;
        text-align: center;
        color: inherit;
        font-size: inherit;
      }

      &__inner {
        padding: 10px 16px;
        border-radius: 4px;
        color: #000;
        font-size: 14px;
        line-height: 20px;
        background: transparent;
        outline: none;
        border: none;
        display: block;
        box-sizing: border-box;
        cursor: pointer;
      }
    }
  }
</style>
