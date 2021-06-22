<template>
    <div class="multi-number-input">
        <!-- https://stackoverflow.com/questions/12374442/chrome-ignores-autocomplete-off?page=1&tab=votes#tab-top -->
        <div style="display: none;">
            <input type="text" id="PreventChromeAutocomplete" name="PreventChromeAutocomplete" autocomplete="address-level4" />
        </div>

        <fragment :key="`single-number-input-${i}`" v-for="(item, i) in numInputs">
            <slot name="separator" v-if="i !== 0"></slot>
            <SingleNumberInput
                :focus="activeInput === i"
                :value="otp[i]"
                :input-classes="inputClasses"
                :input-style="inputStyle"
                :input-wrapper-style="inputWrapperStyle"
                :input-active-wrapper-style="inputActiveWrapperStyle"
                :should-auto-focus="shouldAutoFocus"
                :width="singleWidth"
                :height="singleHeight"
                @on-change="handleOnChange"
                @on-keydown="handleOnKeyDown"
                @on-paste="handleOnPaste"
                @on-focus="handleOnFocus(i)"
                @on-blur="handleOnBlur"
            />
        </fragment>
    </div>
</template>

<script>
import SingleNumberInput from './SingleNumberInput.vue';

// 원본이 된 오픈소스
// https://github.com/bachdgvn/vue-otp-input/blob/develop/src/components/OtpInput.vue

// keyCode constants
const BACKSPACE = 8;
const LEFT_ARROW = 37;
const RIGHT_ARROW = 39;
const DELETE = 46;

export default {
  name: 'MultiNumberInput',
  inheritAttrs: true,
  components: {
    SingleNumberInput,
  },
  props: {
    numInputs: {
      type: Number,
      default: 4,
    },
    singleWidth: {
      type: String,
    },
    singleHeight: {
      type: String,
    },
    inputWrapperStyle: {
      type: Object,
    },
    inputActiveWrapperStyle: {
      type: Object,
    },
    inputClasses: {
      type: String,
    },
    inputStyle: {
      type: [String, Object],
    },
    shouldAutoFocus: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      value: '',
      activeInput: 0,
      otp: [],
      oldOtp: [],
    };
  },
  methods: {
    handleOnFocus(index) {
      this.activeInput = index;
    },
    handleOnBlur() {
      this.activeInput = -1;
    },
    // Helper to return OTP from input
    checkFilledAllInputs() {
      if (this.otp.join('').length === this.numInputs) {
        return this.$emit('on-complete', this.otp.join(''));
      }
      return 'Wait until the user enters the required number of characters';
    },
    // Focus on input by index
    focusInput(input) {
      this.activeInput = Math.max(Math.min(this.numInputs - 1, input), 0);
    },
    // Focus on next input
    focusNextInput() {
      this.focusInput(this.activeInput + 1);
    },
    // Focus on previous input
    focusPrevInput() {
      this.focusInput(this.activeInput - 1);
    },
    // Change OTP value at focused input
    changeCodeAtFocus(value) {
      this.oldOtp = Object.assign([], this.otp);
      this.$set(this.otp, this.activeInput, value);
      if (this.oldOtp.join('') !== this.otp.join('')) {
        this.$emit('on-change', this.otp.join(''));
        this.checkFilledAllInputs();
      }
    },
    // Handle pasted OTP
    handleOnPaste(event) {
      event.preventDefault();
      const pastedData = event.clipboardData
        .getData('text/plain')
        .slice(0, this.numInputs - this.activeInput)
        .split('');
      if (this.inputType === 'number' && !pastedData.join('').match(/^\d+$/)) {
        return 'Invalid pasted data';
      }
      // Paste data from focused input onwards
      const currentCharsInOtp = this.otp.slice(0, this.activeInput);
      const combinedWithPastedData = currentCharsInOtp.concat(pastedData);
      this.$set(this, 'otp', combinedWithPastedData.slice(0, this.numInputs));
      this.focusInput(combinedWithPastedData.slice(0, this.numInputs).length);
      return this.checkFilledAllInputs();
    },
    handleOnChange(value) {
      this.changeCodeAtFocus(value);
      this.focusNextInput();
    },
    clearInput() {
      if (this.otp.length > 0) {
        this.$emit('on-change', '');
      }
      setTimeout(() => {
        this.otp = [];
        this.activeInput = 0;
      }, 0);
    },
    // Handle cases of backspace, delete, left arrow, right arrow
    handleOnKeyDown(event) {
      switch (event.keyCode) {
        case BACKSPACE:
          event.preventDefault();
          this.changeCodeAtFocus('');
          this.focusPrevInput();
          break;
        case DELETE:
          event.preventDefault();
          this.changeCodeAtFocus('');
          break;
        case LEFT_ARROW:
          event.preventDefault();
          this.focusPrevInput();
          break;
        case RIGHT_ARROW:
          event.preventDefault();
          this.focusNextInput();
          break;
        default:
          break;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.multi-number-input {
}
</style>
