<template>
  <div>
    <input
      v-for="(n, index) in code"
      :id="'input_' + index"
      :key="index"
      v-model="code[index]"
      :type="type === 'number' ? 'number' : 'text'"
      pattern="\d*"
      :class="className + ' ' + n"
      :style="
        inputstyle +
        ' ' +
        decideSize() +
        ' text-transform: uppercase; text-align: center;'
      "
      maxlength="1"
      @input="handleInput"
      @keypress="isNumber"
      @keydown.delete="handleDelete"
      @paste="onPaste"
    />
  </div>
</template>

<script setup lang="ts">
import {
  keysAllowedForMix,
  keysAllowedForNumber,
} from "../static/allowedChars";

const props = defineProps({
  fields: { type: Number, required: true },
  inputstyle: { type: String, required: true },
  size: { type: String, required: true },
  type: { type: String, required: true },
  className: { type: String, required: true },
});
const code: string[] = Array(props.fields).fill("");
let dataFromPaste: string[] | undefined;

const emit = defineEmits(["OTPValueChanged"]);
function decideSize() {
  if (props.size === "small") {
    return "width: 50px; height: 30px; font-size: 20px;";
  } else if (props.size === "medium") {
    return "width: 70px; height: 40px; font-size: 30px;";
  } else {
    return "width: 80px; height: 50px; font-size: 40px;";
  }
}

function isNumber(event: Event) {
  (event.currentTarget as HTMLInputElement).value = "";
  const keyPressed: string = (event as KeyboardEvent).key;

  if (props.type === "number") {
    if (!keysAllowedForNumber.includes(keyPressed)) {
      event.preventDefault();
    }
  } else {
    if (!keysAllowedForMix.includes(keyPressed.toUpperCase())) {
      event.preventDefault();
    }
  }
}
function handleInput(event: Event) {
  const inputType = (event as InputEvent).inputType;
  let currentActiveElement = event.target as HTMLInputElement;

  if (inputType === "insertText")
    (currentActiveElement.nextElementSibling as HTMLElement)?.focus();

  if (inputType === "insertFromPaste" && dataFromPaste) {
    for (const num of dataFromPaste) {
      const id: number = parseInt(currentActiveElement.id.split("_")[1]);
      currentActiveElement.value = num;
      code[id] = num;
      if (currentActiveElement.nextElementSibling) {
        currentActiveElement =
          currentActiveElement.nextElementSibling as HTMLInputElement;
        (currentActiveElement.nextElementSibling as HTMLElement)?.focus();
      }
    }
  }
  emit(
    "OTPValueChanged",
    code.map((n) => n.toUpperCase())
  );
}

function handleDelete(event: Event) {
  const value = (event.target as HTMLInputElement).value;
  const currentActiveElement = event.target as HTMLInputElement;
  if (!value)
    (currentActiveElement.previousElementSibling as HTMLElement)?.focus();
}

function onPaste(event: Event) {
  dataFromPaste = (event as ClipboardEvent).clipboardData
    ?.getData("text")
    .trim()
    .split("");

  if (dataFromPaste) {
    for (const num of dataFromPaste) {
      if (!keysAllowedForNumber.includes(num)) event.preventDefault();
    }
  }
}
</script>

<style scoped lang="css">
input {
  width: 150px;
  height: 50px;
  font-size: 40px;
  text-align: center;
  caret-color: transparent !important;
}
/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input {
  -moz-appearance: textfield;
}
</style>