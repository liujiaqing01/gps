<template>
  <div class="table-input-wrap">
    <inner-input/>
  </div>
</template>

<script setup>
import {getCurrentInstance, defineComponent, h, ref} from "vue";

const {proxy} = getCurrentInstance();
import {ElInput, ElDatePicker, ElSelect, ElOption} from "element-plus";

const emit = defineEmits();
const props = defineProps({
  modelValue: [String, Object, Array, Number],
  type: {
    type: String,
    default: 'input',
  },
  disabled: {
    type: Boolean,
    default: false
  },
  attribute: {
    type: Object,
    default: () => {
      return {}
    }
  },
  options: {
    type: Array,
    default: () => {
      return []
    }
  },
  selectLabel: {
    type: String,
    default: 'label'
  },
  selectValue: {
    type: String,
    default: 'value'
  }
});

const show = ref(false)
const Input = ({value, onChange, forwardRef}) => {
  return h(ElInput, {
    ref: forwardRef,
    modelValue: value,
    onInput: onChange,
    placeholder: '请输入',
    ...props.attribute
  })
}

const Date = ({value, onChange, forwardRef}) => {
  return h(ElDatePicker, {
    ref: forwardRef,
    modelValue: value,
    placeholder: '请选择',
    'onUpdate:modelValue': onChange,
    valueFormat: 'YYYY-MM-DD',
    ...props.attribute
  })
}

const Select = ({value, onChange, forwardRef}) => {
  // 生成select下拉数据
  const selectOptions = []
  props.options.forEach(item => {
    selectOptions.push(
        h(ElOption, {
          label: item[props.selectLabel],
          value: item[props.selectValue]
        })
    )
  })
  return h(ElSelect, {
    ref: forwardRef,
    modelValue: value,
    onChange: onChange,
    placeholder: '请选择',
    style: {display: 'block'},
    ...props.attribute
  }, selectOptions)
}

const formObj = {
  'input': Input,
  'inputNumber': Input,
  'date': Date,
  'select': Select
}

const InnerInput = defineComponent({
  setup() {
    const onChange = (value) => {
      if (props.type === 'inputNumber') {
        const precision = props.attribute?.precision
        const min = props.attribute?.min
        value = handleNumber(value, precision, min)
      }
      emit("update:modelValue", value);
      emit("change", value);
    }
    const onEnterEditMode = () => {
      show.value = true
    }
    const onExitEditMode = () => {
      let innerValue = props.modelValue
      show.value = false
      if (props.type === 'inputNumber') {
        const min = props.attribute?.min ?? -Infinity;
        const max = props.attribute?.max ?? Infinity;
        let handleValue = innerValue
        // 如果最后一位是小数点，则删除
        if (String(handleValue).endsWith(".")) {
          handleValue = handleValue.slice(0, handleValue.length - 1);
        }
        if (Number(handleValue) < min) {
          handleValue = min
        }
        if (Number(handleValue) > max) {
          handleValue = max
        }
        handleValue = !Number(handleValue) ? 0 : Number(handleValue)
        if (handleValue !== innerValue) {
          emit("update:modelValue", handleValue);
          emit("handleBlur", handleValue);
        }
      }
      if (props.type === 'input') {
        emit('handleBlur')
      }
    }
    const input = ref()
    const setRef = (el) => {
      input.value = el
      if (el) {
        el.focus?.()
        // 如果为下拉框，则直接将下拉框展示出来
        if (props.type === 'select') {
          el.visible = true
        }
      }
    }
    const getShowValue = () => {
      if (props.type === 'select') {
        const optionsObj = {}
        props.options.forEach(item => {
          optionsObj[item[props.selectValue]] = item[props.selectLabel]
        })
        return optionsObj[props.modelValue]
      } else {
        return props.modelValue
      }
    }
    // 下拉框组件blur事件优先级比change高 会先执行
    const onVisibleChange = (val) => {
      if (!val) {
        // 外传blur事件
        emit('handleBlur')
        show.value = false
      }
    }
    return () => show.value && !props.disabled ? h(formObj[props.type], {
      forwardRef: setRef,
      value: props.modelValue,
      onChange: onChange,
      onBlur: !['select', 'date'].includes(props.type) ? onExitEditMode : null,
      onVisibleChange: ['select', 'date'].includes(props.type) ? onVisibleChange : null,
      onKeydownEnter: onEnterEditMode,
    }) : h('div', {
      class: !props.disabled ? 'editing-trigger' : 'disabled-editing-trigger',
      onClick: onEnterEditMode
    }, getShowValue())
  },
})

/**
 * 处理数字
 * @param num
 * @param precision
 * @param min
 * @returns {*}
 */
function handleNumber(num, precision = 2, min = -Infinity) {
  // 是否可以输入小数
  let needDecimal = precision > 0;
  // 是否可以输入负数
  let needNegative = min < 0
  // 可以输入小数并且可以输入负数
  if (needDecimal && needNegative) {
    // 允许数字、小数点、负号以外的字符
    num = num.replace(/[^\d\.\-]/g, "");
  }
  // 可以输入小数，不可以输入负数
  else if (needDecimal && !needNegative) {
    // 允许数字和小数点以外的字符
    num = num.replace(/[^0-9.]/g, "");
  }
  // 不可以输入小数，也不可以输入负数
  else if (!needDecimal && !needNegative) {
    // 允许数字以外的字符
    num = num.replace(/[^0-9]/g, "");
  }
  // 不可以输入小数，但是可以输入负数
  else if (!needDecimal && needNegative) {
    // 允许数字和负号以外的字符
    num = num.replace(/[^0-9\-]/g, "");
  }
  // 只保留一个小数点
  if (num.indexOf('.') !== num.lastIndexOf('.')) {
    num = num.slice(0, num.lastIndexOf('.')) + num.slice(num.lastIndexOf('.') + 1);
  }
  // 负数只能输入在第一个
  if (num.indexOf('-') !== 0 && num.indexOf('-') !== -1) {
    num = num.slice(0, num.indexOf('-')) + num.slice(num.indexOf('-') + 1);
  }
  // 只保留一个负数符号
  if (num.indexOf('-') !== num.lastIndexOf('-')) {
    num = num.slice(0, num.lastIndexOf('-')) + num.slice(num.lastIndexOf('-') + 1);
  }

  // 控制精度
  const decimalIndex = num.indexOf(".");
  if (decimalIndex !== -1 && num.length > decimalIndex + precision) {
    num = num.slice(0, decimalIndex + precision + 1);
  }
  return num
}

</script>

<style lang="scss" scoped>
.table-input-wrap {
  width: 100%;
}

.disabled-editing-trigger {
  min-width: 100%;
  height: 32px;
  line-height: 32px;
  text-align: center;
  cursor: pointer;
  border-radius: 4px;
  position: relative;
}

.editing-trigger {
  border: 1px var(--el-color-primary) dotted;
  min-width: 100%;
  height: 32px;
  line-height: 32px;
  text-align: center;
  cursor: pointer;
  border-radius: 4px;
  position: relative;

  &::after {
    content: '';
    position: absolute;
    width: 12px;
    height: 12px;
    opacity: 0;
    right: 4px;
    top: 3px;
    transition: opacity 0.5s;
  }
}

.editing-trigger:hover {
  border: 1px var(--el-color-primary) solid;

  &::after {
    background: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAAXNSR0IArs4c6QAAAPZJREFUSEvNlX0RgkAQR9+awAomUBOIEUygUEQjWEDUJF4EIxhBE6zDDTh8HCLcMSP/Mry3+7tlTxj5kZH5/L8gSvUCrBA2JpZ7PRGvDnL4Noc+EdZ1yWBBCf4CHsAcaEgGCSpwIbICxeSSh0lkVkTVW1CLBZSYCXeUGzAFriaR3SBBpXLliLD/5O+AZ+9+7qAeS3aY0UkPJUml8l4dOOFnXbTFUh7Vzg584J0R+cK/CkLAWwWh4E5BSHhDEBruEqgdMWFp5/zHUfx2p1TGNErVCkwiEgLe3oHnNZcV6PyTiw48+TaBXqvCR9i5KnzgnavCF559/wbxpqoZFKctNgAAAABJRU5ErkJggg==') no-repeat;
    background-size: 12px;
    opacity: 1;
  }
}
</style>
