<template>
  <div
    class="mx-datepicker"
    :class="{ 'mx-datepicker-range': range, 'disabled': disabled }"
    :style="{ 'width': computedWidth }"
    v-clickoutside="closePopup">
    <div>
      <input
        class="form-control"
        :class="inputClass"
        ref="input"
        type="text"
        @focus="showPopup"
        @blur="blurPopup"
        :name="inputName"
        :disabled="disabled"
        :readonly="!editable"
        :value="text"
        :placeholder="innerPlaceholder"
        @input="handleInput"
        @change="handleChange">
      <span v-if="!showClearIcon" class="mx-input-append">
        <slot name="calendar-icon">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
            <path d="M 6 1 L 6 3 L 5 3 C 3.9 3 3 3.9 3 5 L 3 19 C 3 20.1 3.9 21 5 21 L 19 21 C 20.1 21 21 20.1 21 19 L 21 5 C 21 3.9 20.1 3 19 3 L 18 3 L 18 1 L 16 1 L 16 3 L 8 3 L 8 1 L 6 1 z M 5 5 L 6 5 L 8 5 L 16 5 L 18 5 L 19 5 L 19 7 L 5 7 L 5 5 z M 5 9 L 19 9 L 19 19 L 5 19 L 5 9 z" fill="#4d4d4d"/>
          </svg>
        </slot>
      </span>
      <span v-else
        class="mx-input-append mx-clear-wrapper"
        @click.stop="clearDate">
        <slot name="mx-clear-icon">
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Capa_1" x="0px" y="0px" viewBox="0 0 212.982 212.982" style="enable-background:new 0 0 212.982 212.982;" width="16px" height="16px"> xml:space="preserve">
            <g id="Close">
              <path style="fill-rule:evenodd;clip-rule:evenodd;" d="M131.804,106.491l75.936-75.936c6.99-6.99,6.99-18.323,0-25.312   c-6.99-6.99-18.322-6.99-25.312,0l-75.937,75.937L30.554,5.242c-6.99-6.99-18.322-6.99-25.312,0c-6.989,6.99-6.989,18.323,0,25.312   l75.937,75.936L5.242,182.427c-6.989,6.99-6.989,18.323,0,25.312c6.99,6.99,18.322,6.99,25.312,0l75.937-75.937l75.937,75.937   c6.989,6.99,18.322,6.99,25.312,0c6.99-6.99,6.99-18.322,0-25.312L131.804,106.491z" fill="#4d4d4d"/>
            </g>
          </svg>
        </slot>
      </span>
    </div>
    <div class="mx-datepicker-popup"
      :style="position"
      v-show="popupVisible"
      @click="unblurPopup"
      ref="calendar">
      <slot name="header">
        <div class="mx-shortcuts-wrapper"
          v-if="range && innnerShortcuts.length">
          <span
            class="mx-shortcuts"
            v-for="(range, index) in innnerShortcuts"
            :key="index"
            @click="selectRange(range)">{{range.text}}</span>
        </div>
      </slot>
      <calendar-panel
        v-if="!range"
        v-bind="$attrs"
        :value="currentValue"
        :visible="popupVisible"
        @select-date="selectDate"
        @select-time="selectTime"></calendar-panel>
      <div class="mx-range-wrapper"
        v-else>
        <calendar-panel
          style="box-shadow:1px 0 rgba(0, 0, 0, .1)"
          v-bind="$attrs"
          :value="currentValue[0]"
          :end-at="currentValue[1]"
          :start-at="null"
          :visible="popupVisible"
          @select-date="selectStartDate"
          @select-time="selectStartTime"></calendar-panel>
        <calendar-panel
          v-bind="$attrs"
          :value="currentValue[1]"
          :start-at="currentValue[0]"
          :end-at="null"
          :visible="popupVisible"
          @select-date="selectEndDate"
          @select-time="selectEndTime"></calendar-panel>
      </div>
      <slot name="footer">
        <div class="mx-datepicker-footer"
          v-if="confirm">
          <button type="button"
            class="mx-datepicker-btn btn btn-sm btn-info"
            @click="confirmDate">{{ confirmText }}</button>
        </div>
      </slot>
    </div>
  </div>
</template>

<script>
import fecha from 'fecha'
import clickoutside from '@/directives/clickoutside'
import { isValidDate, isValidRange, isDateObejct } from '@/utils/index'
import { use, t } from '@/locale/index'
import CalendarPanel from './calendar.vue'

export default {
  name: 'VueDatepicker',
  components: { CalendarPanel },
  directives: {
    clickoutside
  },
  props: {
    value: null,
    placeholder: {
      type: String,
      default: null
    },
    lang: {
      type: [String, Object],
      default: 'en'
    },
    format: {
      type: String,
      default: 'YYYY-MM-DD'
    },
    range: {
      type: Boolean,
      default: false
    },
    rangeSeparator: {
      type: String,
      default: '~'
    },
    width: {
      type: [String, Number],
      default: null
    },
    confirmText: {
      type: String,
      default: 'OK'
    },
    confirm: {
      type: Boolean,
      default: false
    },
    editable: {
      type: Boolean,
      default: true
    },
    disabled: {
      type: Boolean,
      default: false
    },
    clearable: {
      type: Boolean,
      default: true
    },
    shortcuts: {
      type: [Boolean, Array],
      default: true
    },
    inputName: {
      type: String,
      default: 'date'
    },
    inputClass: {
      type: [String, Array],
      default: 'mx-input'
    }
  },
  data () {
    return {
      currentValue: this.range ? [null, null] : null,
      userInput: null,
      popupVisible: false,
      position: {}
    }
  },
  watch: {
    value: {
      immediate: true,
      handler: 'handleValueChange'
    },
    popupVisible (val) {
      if (val) {
        this.initCalendar()
      } else {
        this.userInput = null
      }
    }
  },
  computed: {
    innerPlaceholder () {
      if (typeof this.placeholder === 'string') {
        return this.placeholder
      }
      return this.range ? t('placeholder.dateRange') : t('placeholder.date')
    },
    text () {
      if (this.userInput !== null) {
        return this.userInput
      }
      if (!this.range) {
        return isValidDate(this.value) ? this.stringify(this.value) : ''
      }
      return isValidRange(this.value)
        ? `${this.stringify(this.value[0])} ${this.rangeSeparator} ${this.stringify(this.value[1])}`
        : ''
    },
    computedWidth () {
      if (typeof this.width === 'number' || (typeof this.width === 'string' && /^\d+$/.test(this.width))) {
        return this.width + 'px'
      }
      return this.width
    },
    showClearIcon () {
      return !this.disabled && this.clearable && (this.range ? isValidRange(this.value) : isValidDate(this.value))
    },
    innnerShortcuts () {
      if (Array.isArray(this.shortcuts)) {
        return this.shortcuts
      }
      if (this.shortcuts === false) {
        return []
      }
      const pickers = t('pickers')
      const arr = [
        {
          text: pickers[0],
          start: new Date(),
          end: new Date(Date.now() + 3600 * 1000 * 24 * 7)
        },
        {
          text: pickers[1],
          start: new Date(),
          end: new Date(Date.now() + 3600 * 1000 * 24 * 30)
        },
        {
          text: pickers[2],
          start: new Date(Date.now() - 3600 * 1000 * 24 * 7),
          end: new Date()
        },
        {
          text: pickers[3],
          start: new Date(Date.now() - 3600 * 1000 * 24 * 30),
          end: new Date()
        }
      ]
      return arr
    }
  },
  created () {
    use(this.lang)
  },
  methods: {
    initCalendar () {
      this.handleValueChange(this.value)
      this.displayPopup()
    },
    stringify (date, format) {
      format = format || this.format
      return fecha.format(date, format)
    },
    parseDate (value, format) {
      try {
        format = format || this.format
        return fecha.parse(value, format)
      } catch (e) {
        return false
      }
    },
    dateEqual (a, b) {
      return isDateObejct(a) && isDateObejct(b) && a.getTime() === b.getTime()
    },
    rangeEqual (a, b) {
      return Array.isArray(a) && Array.isArray(b) && a.length === b.length && a.every((item, index) => this.dateEqual(item, b[index]))
    },
    selectRange (range) {
      this.currentValue = [ new Date(range.start), new Date(range.end) ]
      this.updateDate(true)
    },
    clearDate () {
      const date = this.range ? [null, null] : null
      this.currentValue = date
      this.updateDate(true)
    },
    confirmDate () {
      const valid = this.range ? isValidRange(this.currentValue) : isValidDate(this.currentValue)
      if (valid) {
        this.updateDate(true)
      }
      this.$emit('confirm', this.currentValue)
      this.closePopup()
    },
    updateDate (confirm = false) {
      if ((this.confirm && !confirm) || this.disabled) {
        return false
      }
      const equal = this.range ? this.rangeEqual(this.value, this.currentValue) : this.dateEqual(this.value, this.currentValue)
      if (equal) {
        return false
      }
      this.$emit('input', this.currentValue)
      this.$emit('change', this.currentValue)
      return true
    },
    handleValueChange (value) {
      if (!this.range) {
        this.currentValue = isValidDate(value) ? new Date(value) : null
      } else {
        this.currentValue = isValidRange(value) ? [new Date(value[0]), new Date(value[1])] : [null, null]
      }
    },
    selectDate (date) {
      this.currentValue = date
      this.updateDate() && this.closePopup()
    },
    selectStartDate (date) {
      this.$set(this.currentValue, 0, date)
      if (this.currentValue[1]) {
        this.updateDate()
      }
    },
    selectEndDate (date) {
      this.$set(this.currentValue, 1, date)
      if (this.currentValue[0]) {
        this.updateDate()
      }
    },
    selectTime (time) {
      this.currentValue = time
      this.updateDate()
    },
    selectStartTime (time) {
      this.selectStartDate(time)
    },
    selectEndTime (time) {
      this.selectEndDate(time)
    },
    showPopup () {
      if (this.disabled) {
        return
      }
      this.popupVisible = true
    },
    closePopup () {
      this.popupVisible = false
    },
    blurPopup () {
      console.log('blurPopup', this)
    },
    unblurPopup () {
      console.log('unblurPopup', this)
    },
    displayPopup () {
      const dw = document.documentElement.clientWidth
      const dh = document.documentElement.clientHeight
      const InputRect = this.$el.getBoundingClientRect()
      const PopupRect = this.$refs.calendar.getBoundingClientRect()
      this.position = {}
      if (
        dw - InputRect.left < PopupRect.width &&
        InputRect.right < PopupRect.width
      ) {
        this.position.left = 1 - InputRect.left + 'px'
      } else if (InputRect.left + InputRect.width / 2 <= dw / 2) {
        this.position.left = 0
      } else {
        this.position.right = 0
      }
      if (
        InputRect.top <= PopupRect.height + 1 &&
        dh - InputRect.bottom <= PopupRect.height + 1
      ) {
        this.position.top = dh - InputRect.top - PopupRect.height - 1 + 'px'
      } else if (InputRect.top + InputRect.height / 2 <= dh / 2) {
        this.position.top = '100%'
      } else {
        this.position.bottom = '100%'
      }
    },
    handleInput (event) {
      this.userInput = event.target.value
    },
    handleChange (event) {
      const value = event.target.value
      if (this.editable && this.userInput !== null) {
        const calendar = this.$children[0]
        const checkDate = calendar.type === 'date' ? calendar.isDisabledDate : calendar.isDisabledTime
        if (this.range) {
          const range = value.split(` ${this.rangeSeparator} `)
          if (range.length === 2) {
            const start = this.parseDate(range[0], this.format)
            const end = this.parseDate(range[1], this.format)
            if (start && end && !checkDate(start, null, end) && !checkDate(end, start, null)) {
              this.currentValue = [ start, end ]
              this.updateDate(true)
              this.closePopup()
              return
            }
          }
        } else {
          const date = this.parseDate(value, this.format)
          if (date && !checkDate(date, null, null)) {
            this.currentValue = date
            this.updateDate(true)
            this.closePopup()
            return
          }
        }
        this.$emit('input-error', value)
      }
    }
  }
}
</script>
