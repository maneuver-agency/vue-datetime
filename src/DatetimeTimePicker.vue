<template>
  <div
    :class="{
      'vdatetime-time-picker': true,
      'vdatetime-time-picker__with-suffix': use12Hour,
    }"
  >
    <div
      class="vdatetime-time-picker__list vdatetime-time-picker__list--hours"
      ref="hourList"
    >
      <div
        class="vdatetime-time-picker__item tetten"
        v-for="hour in hours"
        @click="selectHour(hour)"
        :class="{
          'vdatetime-time-picker__item--selected': hour.selected,
          'vdatetime-time-picker__item--disabled': hour.disabled,
        }"
      >
        <div>{{ formatHour(hour.number) }}</div>
        <div
          class="vdatetime-time-picker__item__disabled-label"
          v-if="hour.label"
        >
          {{ hour.label.value }}
        </div>
      </div>
    </div>
    <div
      class="vdatetime-time-picker__list vdatetime-time-picker__list--minutes"
      ref="minuteList"
      v-if="minuteStep < 60"
    >
      <div
        class="vdatetime-time-picker__item"
        v-for="minute in minutes"
        @click="selectMinute(minute)"
        :class="{
          'vdatetime-time-picker__item--selected': minute.selected,
          'vdatetime-time-picker__item--disabled': minute.disabled,
        }"
      >
        {{ minute.number }}
      </div>
    </div>
    <div
      class="vdatetime-time-picker__list vdatetime-time-picker__list--suffix"
      ref="suffixList"
      v-if="use12Hour"
    >
      <div
        class="vdatetime-time-picker__item"
        @click="selectSuffix('am')"
        :class="{ 'vdatetime-time-picker__item--selected': hour < 12 }"
      >
        am
      </div>
      <div
        class="vdatetime-time-picker__item"
        @click="selectSuffix('pm')"
        :class="{ 'vdatetime-time-picker__item--selected': hour >= 12 }"
      >
        pm
      </div>
    </div>
  </div>
</template>

<script>
import { hours, minutes, pad, timeComponentIsDisabled } from './util'

export default {
  props: {
    hour: {
      type: Number,
      required: true
    },
    minute: {
      type: Number,
      required: true
    },
    use12Hour: {
      type: Boolean,
      default: false
    },
    hourStep: {
      type: Number,
      default: 1
    },
    minuteStep: {
      type: Number,
      default: 1
    },
    minTime: {
      type: String,
      default: null
    },
    maxTime: {
      type: String,
      default: null
    },
    hourRange: {
      type: Array,
      default () {
        return []
      }
    },
    hourLabels: {
      type: Array,
      default () {
        return []
      }
    },
    disabledHours: {
      type: Array,
      default () {
        return []
      }
    }
  },

  computed: {
    hours () {
      return hours(this.hourStep, this.hourRange)
        .filter((hour) => {
          if (!this.use12Hour) {
            return true
          } else {
            if (this.hour < 12) {
              return hour < 12
            } else {
              return hour >= 12
            }
          }
        })
        .map((hour) => {
          return {
            number: pad(hour),
            selected: hour === this.hour,
            disabled:
              timeComponentIsDisabled(this.minHour, this.maxHour, hour) ||
              this.disabledHours.includes(hour),
            label: this.hourLabels.find((h) => h.hour === hour)
          }
        })
    },
    minutes () {
      return minutes(this.minuteStep).map((minute) => ({
        number: pad(minute),
        selected: minute === this.minute,
        disabled: timeComponentIsDisabled(
          this.minMinute,
          this.maxMinute,
          minute
        )
      }))
    },
    minHour () {
      return this.minTime ? parseInt(this.minTime.split(':')[0]) : null
    },
    minMinute () {
      return this.minTime && this.minHour === this.hour
        ? parseInt(this.minTime.split(':')[1])
        : null
    },
    maxHour () {
      return this.maxTime ? parseInt(this.maxTime.split(':')[0]) : null
    },
    maxMinute () {
      return this.maxTime && this.maxHour === this.hour
        ? parseInt(this.maxTime.split(':')[1])
        : null
    }
    // hourLabels() {
    //   return [
    //     { hour: 8, value: "100/250 beschikbaar" },
    //     { hour: 9, value: "volzet" },
    //     { hour: 10, value: "volzet" },
    //   ];
    // },
    // disabledHours() {
    //   return [9, 10];
    // },
  },

  methods: {
    selectHour (hour) {
      if (hour.disabled) {
        return
      }

      this.$emit('change', { hour: parseInt(hour.number) })
    },
    selectMinute (minute) {
      if (minute.disabled) {
        return
      }

      this.$emit('change', { minute: parseInt(minute.number) })
    },
    selectSuffix (suffix) {
      if (suffix === 'am') {
        if (this.hour >= 12) {
          this.$emit('change', {
            hour: parseInt(this.hour - 12),
            suffixTouched: true
          })
        }
      }
      if (suffix === 'pm') {
        if (this.hour < 12) {
          this.$emit('change', {
            hour: parseInt(this.hour + 12),
            suffixTouched: true
          })
        }
      }
    },
    formatHour (hour) {
      const numHour = Number(hour)
      if (this.use12Hour) {
        if (numHour === 0) {
          return 12
        }
        if (numHour > 12) {
          return numHour - 12
        }
        return numHour
      }

      if (this.minuteStep === 60) {
        return `${hour}:00`
      }

      return hour
    }
  },

  mounted () {
    const selectedHour = this.$refs.hourList.querySelector(
      '.vdatetime-time-picker__item--selected'
    )
    this.$refs.hourList.scrollTop = selectedHour
      ? selectedHour.offsetTop - 250
      : 0

    if (this.$refs.minuteList) {
      const selectedMinute = this.$refs.minuteList.querySelector(
        '.vdatetime-time-picker__item--selected'
      )
      this.$refs.minuteList.scrollTop = selectedMinute
        ? selectedMinute.offsetTop - 250
        : 0
    }
  }
}
</script>

<style>
.vdatetime-time-picker {
  box-sizing: border-box;
  display: flex;

  &::after {
    content: "";
    display: table;
    clear: both;
  }

  & * {
    box-sizing: border-box;
  }
}

.vdatetime-time-picker__list {
  /* float: left;
  width: 50%; */
  flex: 1 0 33.33%;
  height: 305px;
  overflow-y: scroll;

  &::-webkit-scrollbar {
    width: 3px;
  }

  &::-webkit-scrollbar-track {
    background: #efefef;
  }

  &::-webkit-scrollbar-thumb {
    background: #ccc;
  }
}

/* .vdatetime-time-picker__with-suffix .vdatetime-time-picker__list {
  width: 33.3%;
} */

.vdatetime-time-picker__item {
  padding: 10px 0;
  font-size: 20px;
  text-align: center;
  cursor: pointer;
  transition: font-size 0.3s;
}

.vdatetime-time-picker__item:hover {
  font-size: 32px;
}

.vdatetime-time-picker__item--selected {
  color: #3f51b5;
  font-size: 32px;
}

.vdatetime-time-picker__item--disabled {
  opacity: 0.4;
  cursor: default;
  font-size: 20px !important;
}
.vdatetime-time-picker__item__disabled-label {
  font-size: 10px;
}
</style>
