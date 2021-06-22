<template>
    <div class="single-number-input">
        <div :style="{ display: 'inline-block', width: width, height: height }">
            <div class="input-wrapper" :style="{ ...(value ? inputActiveWrapperStyle : inputWrapperStyle) }">
                <div class="back-text">{{model ? '*' : ''}}</div>
                <input
                    ref="input"
                    type="text"
                    pattern="[0-9]"
                    inputmode="numeric"
                    maxlength="1"
                    :style="inputStyle"
                    autocomplete="off"
                    v-model="model"
                    :class="inputClasses"
                    @input="handleOnChange"
                    @keydown="handleOnKeyDown"
                    @paste="handleOnPaste"
                    @focus="handleOnFocus"
                    @blur="handleOnBlur"
                >
            </div>
        </div>
    </div>
</template>

<script>
export default {
  name: 'SingleNumberInput',
  inheritAttrs: true,
  components: {
  },
  props: {
    width: {
      type: String,
      default() {
        return '48.5px';
      },
    },
    height: {
      type: String,
      default() {
        return '64px';
      },
    },
    value: {
      type: String,
    },
    inputClasses: {
      type: String,
    },
    inputWrapperStyle: {
      type: Object,
      default() {
        return {};
      },
    },
    inputActiveWrapperStyle: {
      type: Object,
      default() {
        return {};
      },
    },
    inputStyle: {
      type: [String, Object],
    },
    shouldAutoFocus: {
      type: Boolean,
    },
    focus: {
      type: Boolean,
    },
  },
  data() {
    return {
      model: this.value || '',
    };
  },
  watch: {
    value(newValue, oldValue) {
      if (newValue !== oldValue) {
        this.model = newValue;
      }
    },
    // whenever question changes, this function will run
    focus(newFocusValue, oldFocusValue) {
      // Check if focusedInput changed
      // Prevent calling function if input already in focus
      if (oldFocusValue !== newFocusValue && (this.$refs.input && this.focus)) {
        this.$refs.input.focus();
        this.$refs.input.select();
      }
    },
  },
  created() {
  },
  mounted() {
    if (this.$refs.input && this.focus && this.shouldAutoFocus) {
      setTimeout(() => this.$refs.input.focus(), 0);
    }
  },
  methods: {
    handleOnChange() {
      if (this.model.length > 1) {
        this.model = this.model.slice(0, 1);
      }
      return this.$emit('on-change', this.model);
    },
    handleOnKeyDown(event) {
      // Only allow characters 0-9, DEL, Backspace and Pasting
      const keyEvent = (event) || window.event;
      const charCode = (keyEvent.which) ? keyEvent.which : keyEvent.keyCode;
      if (this.isCodeNumeric(charCode)
            || (charCode === 8) || (charCode === 37) || (charCode === 39)
            || (charCode === 86)
            || (charCode === 46)) {
        this.$emit('on-keydown', event);
      } else {
        keyEvent.preventDefault();
      }
    },
    isCodeNumeric(charCode) {
      // numeric keys and numpad keys
      return (charCode >= 48 && charCode <= 57) || (charCode >= 96 && charCode <= 105);
    },
    handleOnPaste(event) {
      return this.$emit('on-paste', event);
    },
    handleOnFocus() {
      this.$refs.input.select();
      return this.$emit('on-focus');
    },
    handleOnBlur() {
      return this.$emit('on-blur');
    },

  },
};
</script>

<style lang="scss" scoped>
.single-number-input {
    display: inline-block;
    input {
        position: absolute;
        color: transparent;
        caret-color: black;
        background: transparent;
        border: none;
        outline: none;
        width: 100%;
        height: 100%;
        text-align: center;
    }

    .input-wrapper {
        position: relative;
        background: white;
        border: 1px solid #ECECED;
        width: 100%;
        height: 100%;
    }

    .back-text {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        align-items: center;
        position: absolute;
        width: 100%;
        height: 100%;
    }
}
</style>
