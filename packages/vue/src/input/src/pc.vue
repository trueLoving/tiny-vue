<!--
 * Copyright (c) 2022 - present TinyVue Authors.
 * Copyright (c) 2022 - present Huawei Cloud Computing Technologies Co., Ltd.
 *
 * Use of this source code is governed by an MIT-style license.
 *
 * THE OPEN SOURCE SOFTWARE IN THIS PRODUCT IS DISTRIBUTED IN THE HOPE THAT IT WILL BE USEFUL,
 * BUT WITHOUT ANY WARRANTY, WITHOUT EVEN THE IMPLIED WARRANTY OF MERCHANTABILITY OR FITNESS FOR
 * A PARTICULAR PURPOSE. SEE THE APPLICABLE LICENSES FOR MORE DETAILS.
 *
 -->
<template>
  <div
    :class="[
      $attrs.class,
      type === 'textarea' ? 'tiny-textarea' : 'tiny-input',
      state.inputSize ? 'tiny-input-' + state.inputSize : '',
      {
        'is-disabled': state.inputDisabled,
        'is-exceed': state.inputExceed,
        'tiny-input-group': slots.prepend || slots.append,
        'tiny-input-group-append': slots.append,
        'tiny-input-group-prepend': slots.prepend,
        'tiny-input-prefix': slots.prefix || prefixIcon,
        'tiny-input-suffix': slots.suffix || suffixIcon || clearable || showPassword || (mask && state.inputDisabled),
        'tiny-input-word-limit': state.isWordLimitVisible,
        'is-display-only': state.isDisplayOnly
      }
    ]"
    :style="$attrs.style"
    @mouseenter="
      ($event) => {
        state.hovering = true
        $emit('mouseenter', $event)
      }
    "
    @mouseleave="
      ($event) => {
        state.hovering = false
        $emit('mouseleave', $event)
      }
    "
    @click="$emit('click', $event)"
  >
    <template v-if="type !== 'textarea'">
      <!-- 前置元素 -->
      <div class="tiny-input-group__prepend" ref="prepend" v-if="slots.prepend">
        <slot name="prepend"></slot>
      </div>
      <span class="tiny-input-display-only">
        <tiny-tooltip
          v-if="state.isDisplayOnly"
          effect="light"
          :content="state.displayOnlyTooltip"
          :display="type === 'password'"
          placement="top"
          @mouseenter.native="handleEnterDisplayOnlyContent"
        >
          <span class="tiny-input-display-only__content" v-if="type === 'password'">{{ state.hiddenPassword }}</span>
          <span class="tiny-input-display-only__mask" v-else-if="mask">
            <span class="tiny-input-display-only__mask-text">{{ state.displayedMaskValue }}</span>
            <component
              :is="state.maskValueVisible ? 'icon-eyeopen' : 'icon-eyeclose'"
              @click.native="state.maskValueVisible = !state.maskValueVisible"
            ></component>
          </span>
          <span class="tiny-input-display-only__content" v-else>{{ state.displayOnlyText }}</span>
        </tiny-tooltip>
        <input
          v-if="type !== 'textarea'"
          ref="input"
          :name="name"
          data-tag="tiny-input-inner"
          v-bind="a($attrs, ['type', 'class', 'style', '^on[A-Z]', 'id'])"
          :class="['tiny-input__inner', mask && state.inputDisabled && !state.maskValueVisible && 'tiny-input__mask']"
          :tabindex="tabindex"
          :type="showPassword ? (state.passwordVisible ? 'text' : 'password') : type"
          :disabled="state.inputDisabled"
          :readonly="readonly"
          :unselectable="readonly ? 'on' : 'off'"
          :autocomplete="autocomplete"
          @compositionend="handleCompositionEnd"
          @compositionupdate="handleCompositionUpdate"
          @compositionstart="handleCompositionStart"
          @blur="handleBlur"
          @focus="handleFocus"
          @input="handleInput"
          @change="handleChange"
          :aria-label="label"
          @keyup="$emit('keyup', $event)"
          @keydown="$emit('keydown', $event)"
          @paste="$emit('paste', $event)"
        />
      </span>
      <tiny-tall-storage
        v-if="isMemoryStorage"
        :name="name"
        :localstorage-data="storageData"
        :is-memory-storage="isMemoryStorage"
        @selected="selectedMemory"
      ></tiny-tall-storage>
      <!-- 前置内容 -->
      <span class="tiny-input__prefix" ref="prefix" v-if="!state.isDisplayOnly && (slots.prefix || prefixIcon)">
        <slot name="prefix"></slot>
        <component v-if="prefixIcon" :is="prefixIcon" class="tiny-svg-size tiny-input__icon" />
      </span>
      <!-- 后置内容 -->
      <div>
        <transition name="tiny-transition-icon-out-in">
          <span class="tiny-input__suffix" ref="suffix" v-if="!state.isDisplayOnly && getSuffixVisible()">
            <span class="tiny-input__suffix-inner">
              <!-- 清除按钮前置 -->
              <icon-close
                v-if="frontClearIcon && state.showClear"
                class="tiny-svg-size tiny-input__icon tiny-input__clear"
                @mousedown.prevent
                @click="clear"
              ></icon-close>
              <template v-if="!state.showClear || !state.showPwdVisible || !state.isWordLimitVisible">
                <slot name="suffix"></slot>
                <component v-if="suffixIcon" :is="suffixIcon" class="tiny-svg-size tiny-input__icon" />
              </template>
              <!-- 清除按钮后置 -->
              <icon-close
                v-if="!frontClearIcon && state.showClear"
                class="tiny-svg-size tiny-input__icon tiny-input__clear"
                @mousedown.prevent
                @click="clear"
              ></icon-close>
              <component
                v-if="showPassword"
                :is="state.passwordVisible ? 'icon-eyeopen' : 'icon-eyeclose'"
                class="tiny-svg-size tiny-input__icon"
                @click.native="handlePasswordVisible"
              ></component>
              <component
                v-if="mask && state.inputDisabled"
                :is="state.maskValueVisible ? 'icon-eyeopen' : 'icon-eyeclose'"
                class="tiny-svg-size tiny-input__icon"
                @click.native="state.maskValueVisible = !state.maskValueVisible"
              ></component>
              <span v-if="state.isWordLimitVisible" class="tiny-input__count">
                <span class="tiny-input__count-inner"
                  ><span class="tiny-input__count-text-length">
                    {{ state.showWordLimit ? `${state.textLength}` : '' }}
                  </span>
                  <span class="tiny-input__count-upper-limit">
                    {{ state.showWordLimit ? `/${state.upperLimit}` : state.textLength }}
                  </span></span
                >
              </span>
            </span>
            <i class="tiny-input__icon" v-if="state.validateState" :class="['tiny-input__validateIcon', validateIcon]">
            </i>
          </span>
        </transition>
      </div>
      <!-- 后置元素 -->
      <div class="tiny-input-group__append" ref="append" v-if="slots.append">
        <slot name="append"></slot>
      </div>
      <!-- Panel弹窗（例如时间组件的非范围选择窗口） -->
      <div class="tiny-input-group__panel" ref="panel" v-if="slots.panel">
        <slot name="panel"></slot>
      </div>
    </template>
    <span
      :class="[
        autosize ? 'tiny-textarea-autosize-display-only' : 'tiny-textarea-display-only',
        hoverExpand && 'tiny-textarea__inner-con'
      ]"
      v-else
    >
      <tiny-tooltip
        v-if="state.isDisplayOnly"
        effect="light"
        :content="state.displayOnlyTooltip"
        placement="top"
        @mouseenter.native="handleEnterDisplayOnlyContent($event, 'textarea')"
      >
        <span class="tiny-textarea-display-only__content">{{ state.displayOnlyText }}</span>
      </tiny-tooltip>
      <textarea
        ref="textarea"
        v-bind="a($attrs, ['type', 'class', 'style', 'id'])"
        :tabindex="tabindex"
        class="tiny-textarea__inner"
        :class="hoverExpand && !state.enteredTextarea && 'tiny-textarea__fix-height'"
        @compositionstart="handleCompositionStart"
        @compositionupdate="handleCompositionUpdate"
        @compositionend="handleCompositionEnd"
        @input="handleInput"
        :disabled="state.inputDisabled"
        :readonly="readonly"
        :unselectable="readonly ? 'on' : 'off'"
        :autocomplete="autocomplete"
        :style="state.textareaStyle"
        @focus="handleFocus"
        @blur="handleBlur"
        @change="handleChange"
        @mouseenter="handleEnterTextarea($event)"
        @mouseleave="handleLeaveTextarea($event)"
        :aria-label="label"
        @keyup="$emit('keyup', $event)"
        @keydown="$emit('keydown', $event)"
        @paste="$emit('paste', $event)"
      >
      </textarea>
    </span>
    <span v-if="state.isWordLimitVisible && type === 'textarea'" class="tiny-input__count">
      <span class="tiny-input__count-inner"
        ><span class="tiny-input__count-text-length">
          {{ state.showWordLimit ? `${state.textLength}` : '' }}
        </span>
        <span class="tiny-input__count-upper-limit">
          {{ state.showWordLimit ? `/${state.upperLimit}` : state.textLength }}
        </span>
      </span>
    </span>
    <slot></slot>
  </div>
</template>

<script>
import { renderless, api } from '@opentiny/vue-renderless/input/vue'
import { props, setup, defineComponent } from '@opentiny/vue-common'
import TinyTallStorage from './tall-storage.vue'
import { IconClose, IconEyeopen, IconEyeclose } from '@opentiny/vue-icon'
import Tooltip from '@opentiny/vue-tooltip'
import '@opentiny/vue-theme/input/index.less'
import '@opentiny/vue-theme/textarea/index.less'

export default defineComponent({
  emits: [
    'update:modelValue',
    'change',
    'clear',
    'focus',
    'blur',
    'keyup',
    'keydown',
    'paste',
    'mouseenter',
    'mouseleave',
    'click',
    'input'
  ],
  components: {
    IconClose: IconClose(),
    IconEyeopen: IconEyeopen(),
    IconEyeclose: IconEyeclose(),
    TinyTallStorage,
    TinyTooltip: Tooltip
  },

  props: [
    ...props,
    'name',
    'size',
    'form',
    'type',
    'mask',
    'label',
    'modelValue',
    'height',
    'resize',
    'counter',
    'autosize',
    'disabled',
    'readonly',
    'tabindex',
    'clearable',
    'suffixIcon',
    'prefixIcon',
    'autocomplete',
    'showPassword',
    'memorySpace',
    'validateEvent',
    'showWordLimit',
    'displayOnly',
    'displayOnlyContent',
    'frontClearIcon',
    'showEmptyValue',
    'hoverExpand'
  ],
  setup(props, context) {
    return setup({ props, context, renderless, api })
  }
})
</script>
