<template>
  <div class="d-flex daterangepicker-row">
    <!-- Calendars -->
    <div class="daterangepicker-col" v-for="calendarIndex in calendarCount" :key="calendarIndex">
      <date-range-picker-calendar
        :calendarIndex="calendarIndex"
        :calendarCount="calendarCount"
        :month="month"
        :startDate="startDate"
        :endDate="endDate"
        :compare="compare"
        :startDateCompare="startDateCompare"
        :endDateCompare="endDateCompare"
        :step="step"
        v-on:goToPrevMonth="goToPrevMonth"
        v-on:goToNextMonth="goToNextMonth"
        v-on:selectDate="selectDate"
        v-on:nextStep="nextStep"
      />
    </div>

    <!-- Right form -->
    <div class="daterangepicker-col">
      <div class="form-group">
        <select class="custom-select" :class="compare ? 'daterangepicker-range-border' : ''" v-model="rangeSelect">
          <option value="custom">Custom range</option>
          <option v-for="(range, rangeKey) in ranges" :key="rangeKey" :value="rangeKey">{{ range.label }}</option>
        </select>
      </div>
      <div class="form-group form-inline flex-nowrap">
        <input type="text" class="form-control w-100 daterangepicker-date-input"
          ref="startDate"
          :value="startDate | dateFormat"
          @focus="step = 'selectStartDate'" @blur="inputDate"
        >
        <span class="mx-2">
        <font-awesome-icon icon="caret-right" fixed-width />
        </span>
        <input type="text" class="form-control w-100 daterangepicker-date-input"
          ref="endDate"
          :value="endDate | dateFormat"
          @focus="step = 'selectEndDate'" @blur="inputDate"
        >
      </div>
      <div class="form-group" v-if="allowCompare">
        <div class="custom-control custom-checkbox">
          <input type="checkbox" class="custom-control-input" :id="'date-range-picker-compare-' + _uid" v-model="compare">
          <label class="custom-control-label" :for="'date-range-picker-compare-' + _uid">Compare</label>
        </div>
      </div>
      <div v-if="compare">
        <div class="form-group">
          <select class="custom-select" :class="compare ? 'daterangepicker-range-border compare' : ''" v-model="rangeSelectCompare">
            <option value="custom">Custom range</option>
            <option v-for="(compareRange, compareRangeKey) in compareRanges" :key="compareRangeKey" :value="compareRangeKey">{{ compareRange.label }}</option>
          </select>
        </div>
        <div class="form-group form-inline flex-nowrap">
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
            ref="startDateCompare"
            :value="startDateCompare | dateFormat"
            @focus="step = 'selectStartDateCompare'" @blur="inputDate"
            @keyup.enter="inputDate"
          >
          <span class="mx-2">
          <font-awesome-icon icon="caret-right" fixed-width />
          </span>
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
            ref="endDateCompare"
            :value="endDateCompare | dateFormat"
            @focus="step = 'selectEndDateCompare'" @blur="inputDate"
            @keyup.enter="inputDate"
          >
        </div>
      </div>
      <div class="form-group form-inline justify-content-end mb-0">
        <button type="button" class="btn btn-light" @click="cancel">Cancel</button>
        <button type="button" class="btn btn-warning ml-2" @click="submit">Submit</button>
      </div>
    </div>
  </div>
</template>

<script>
import DateRangePickerCalendar from './DateRangePickerCalendar'
import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
import fontawesome from '@fortawesome/fontawesome'
import faCaretRight from '@fortawesome/fontawesome-free-solid/faCaretRight'
import moment from 'moment'

fontawesome.library.add(faCaretRight)

export default {
  props: {
    calendarCount: {
      type: Number,
      default: 2
    },
    allowCompare: {
      type: Boolean,
      default: true
    },
    ranges: {
      type: Object,
      default: function() {
        return {
          today: {
            label: 'Today',
            startDate: moment().startOf('day'),
            endDate: moment().endOf('day')
          },
          yesterday: {
            label: 'Yesterday',
            startDate: moment().subtract(1, 'day').startOf('day'),
            endDate: moment().subtract(1, 'day').endOf('day')
          },
          lastWeek: {
            label: 'Last week',
            startDate: moment().subtract(7, 'days').day('Monday').subtract(1,'day').startOf('day'),
            endDate: moment().subtract(7, 'days').day('Saturday').endOf('day')
          },
          lastMonth: {
            label: 'Last month',
            startDate: moment().subtract(1, 'month').startOf('month'),
            endDate: moment().subtract(1, 'month').endOf('month').startOf('day')
          },
          lastSevenDays: {
            label: 'Last 7 days',
            startDate: moment().subtract(7,'days').startOf('day'),
            endDate: moment().subtract(1,'day').endOf('day')
          },
          lastThirtyDays: {
            label: 'Last 30 days',
            startDate: moment().subtract(30,'days').startOf('day'),
            endDate: moment().subtract(1,'day').endOf('day')
          },
         
        }
      }
    },
    compareRanges: {
      type: Object,
      default: function() {
        return {
          previousPeriod: {
            label: 'Previous period',
            startDate: moment().subtract(1,'day').startOf('day'),
            endDate: moment().subtract(1,'day').endOf('day'),
            processDate: function(start,end,rangeSelect) {
              let nDays = 0;
              if (rangeSelect == 'lastMonth') {
                nDays =moment(start).subtract(1,'month').daysInMonth();
              }
              else if (rangeSelect == 'custom'){
                const checkDiff =moment.duration(end.diff(start)).asDays();
                nDays = checkDiff>1 || !end.isSame(start,'day') ? checkDiff+1 :1;
              }
              else {
                 nDays =moment.duration(end.diff(start)).asDays();
              }
              
              
            
              return {
                startDate: moment(start).subtract(nDays,'day').startOf('day'),
                endDate: moment(start).subtract(1,'day').endOf('day'),
              }
            }
          },
          previousYear: {
            label: 'Previous year',
            startDate: moment().subtract(1, 'year').startOf('day'),
            endDate: moment().subtract(1, 'year').endOf('day'),
            processDate: function(start,end,rangeSelect) {
              return {
                startDate: moment(start).subtract(1,'year').startOf('day'),
                endDate: moment(end).subtract(1,'year').endOf('day'),
              }
            }
          }
        }
      }
    },
    defaultRangeSelect: {
      type: String,
      default: 'today'
    },
    defaultRangeSelectCompare: {
      type: String,
      default: 'previousPeriod'
    }
  },
  data: () => {
    return {
      startDate: moment(),
      endDate: moment(),
      startDateCompare: moment(),
      endDateCompare: moment(),
      rangeSelect: null,
      rangeSelectCompare: null,
      compare: false,
      month: moment().subtract(1, 'month').startOf('month'),
      step: null
    }
  },
  computed: {
    nextMonth: function() {
      return moment(this.month).add(1, 'month')
    },
    // For multi prop watchers
    range: function() {
      return this.startDate, this.endDate
    },
    rangeCompare: function() {
      return this.startDateCompare, this.endDateCompare
    }
  },
  methods: {
    goToPrevMonth: function() {
      this.month = moment(this.month).subtract(1, 'month')
    },
    goToNextMonth: function() {
      this.month = moment(this.month).add(1, 'month')
    },
    selectRange: function(rangeKey) {
      let predefinedRange = false

      // Predefined ranges
      for (const _rangeKey of Object.keys(this.ranges)) {
        const range = this.ranges[_rangeKey]
        if (rangeKey == _rangeKey) {
          predefinedRange = true

          if (!this.startDate.isSame(range.startDate)) {
            this.startDate = moment(range.startDate)
          }
          if (!this.endDate.isSame(range.endDate)) {
            this.endDate = moment(range.endDate)
          }
        }
      }

      // Custom range
      if (!predefinedRange && this.step == null) {
        this.step = 'selectStartDate'
        this.$refs.startDate.focus()
      }

      if(this.step=='selectEndDate') {
        this.step = 'selectStartDate';
      } else if(this.step == 'selectEndDateCompare') {
        this.step = 'selectStartDateCompare';
      }
      if(this.compare){
        this.selectRangeCompare(this.rangeSelectCompare);
      }
    },
    selectRangeCompare: function(rangeKey) {
      let predefinedRange = false

      // Predefined ranges
      for (const _rangeKey of Object.keys(this.compareRanges)) {
        const range = this.compareRanges[_rangeKey]
        if (rangeKey == _rangeKey) {
          predefinedRange = true
          const resultDate = range.processDate(this.startDate,this.endDate,this.rangeSelect);
          this.compareRanges[rangeKey] = Object.assign(this.compareRanges[rangeKey],resultDate);
          if (!this.startDateCompare.isSame(resultDate.startDate)) {
            this.startDateCompare = moment(resultDate.startDate)
          }
          if (!this.endDateCompare.isSame(resultDate.endDate)) {
            this.endDateCompare = moment(resultDate.endDate)
          }
        }
      }
      
      if(this.step=='selectEndDate') {
        this.step = 'selectStartDate';
      } else if(this.step == 'selectEndDateCompare') {
        this.step = 'selectStartDateCompare';
      }
      // Custom range
      if (!predefinedRange && this.step == null && this.compare) {
        this.step = 'selectStartDateCompare'
        this.$refs.startDateCompare.focus()
      }
    },
    selectDate: function(date) {
      if (this.step == 'selectStartDate') {
        //this.endDate = date;
        this.startDate = date;

      } else if (this.step == 'selectEndDate') {
        this.endDate = date
      } else if (this.step == 'selectStartDateCompare') {
        this.startDateCompare = date
      } else if (this.step == 'selectEndDateCompare') {
        this.endDateCompare = date
      }else {
        if(this.compare && this.rangeSelectCompare === 'custom'){
          this.$refs.startDateCompare.focus();
        }else
        {
          this.$refs.startDate.focus();
        }
        
      }
    
    },
    // Step flow for date range selections
    nextStep: function() {
      if(this.step == null) {
        this.step == 'selectStartDate';
        this.$refs.startDate.focus();
      }
      else if (this.step == 'selectStartDate') {
        this.step = 'selectEndDate'
        this.$refs.endDate.focus()
      } else if (this.step == 'selectEndDate') {
        this.step = null
        if(this.compare){
          this.selectRangeCompare(this.rangeSelectCompare);
        }
        this.$refs.endDate.blur()
      } else if (this.step == 'selectStartDateCompare') {
        this.step = 'selectEndDateCompare'
        this.$refs.endDateCompare.focus()
      } else if (this.step == 'selectEndDateCompare') {
        this.step = null
        this.$refs.endDateCompare.blur()
        this.$refs.startDate.focus();
      }
    },
    // Try to update the step date from an input value
    inputDate: function(input) {
      let date = moment(input.target.value, 'YYYY-MM-DD')
      if (date.isValid()) {
        this.selectDate(date)
      }
      if(this.step == 'endDate' || this.step== 'endDateCompare') {
        this.nextStep();
      }
      
    },
    // Submit button
    submit: function() {
      this.$emit('submit', {
        startDate: this.startDate,
        endDate: this.endDate,
        compare: this.compare,
        startDateCompare: this.startDateCompare,
        endDateCompare: this.endDateCompare
      })
    },
    // Cancel button
    cancel: function() {
      this.$emit('cancel')
    }
  },
  watch: {
    compare : function(newVal) {
      //reset
      if(this.step){
        this.nextStep();
      }
      if(newVal === true) {
        this.selectRangeCompare(this.rangeSelectCompare);
      }
      
    },
    rangeSelect: function(rangeKey) {
      this.selectRange(rangeKey)
    },
    rangeSelectCompare: function(rangeKey) {
      this.selectRangeCompare(rangeKey)
    },
    range: function() {
      let predefinedRange = false

      // Predefined ranges
      for (const rangeKey of Object.keys(this.ranges)) {
        const range = this.ranges[rangeKey]
        if (this.startDate.isSame(range.startDate) && this.endDate.isSame(range.endDate)) {
          predefinedRange = true
          if (this.rangeSelect != rangeKey) {
            this.rangeSelect = rangeKey
          }
        }
      }

      // Custom range
      if (!predefinedRange) {
        if (this.rangeSelect != 'custom') {
          this.rangeSelect = 'custom'
        }
      }
    },
    rangeCompare: function() {
      let predefinedRange = false

      // Predefined ranges
      for (const rangeKey of Object.keys(this.compareRanges)) {
        const range = this.compareRanges[rangeKey]
        if (this.startDateCompare.isSame(range.startDate) && this.endDateCompare.isSame(range.endDate)) {
          predefinedRange = true
          if (this.rangeSelectCompare != rangeKey) {
            this.rangeSelectCompare = rangeKey
          }
        }
      }

      // Custom range
      if (!predefinedRange) {
        if (this.rangeSelectCompare != 'custom') {
          this.rangeSelectCompare = 'custom'
        }
      }
    }
  },
  filters: {
    dateFormat: function(value) {
      return value ? value.format('YYYY-MM-DD') : ''
    }
  },
  created: function() {
    // Initialize ranges
    this.rangeSelect = this.defaultRangeSelect
    this.rangeSelectCompare = this.defaultRangeSelectCompare
  },
  components: { DateRangePickerCalendar, FontAwesomeIcon }
}
</script>

<style>
/* Custom row */
.daterangepicker-row {
  margin: -0.5rem;
}

.daterangepicker-col {
  padding: 0.5rem;
  flex-basis: 100%;
}

/* Make sure that the full date (YYYY-MM-DD) is displayed */
.daterangepicker-date-input {
  min-width: 120px;
}

/* Select menus border */
.daterangepicker-range-border {
  border-color: #17a2b8 !important;
}

.daterangepicker-range-border.compare {
  border-color: #ff9307 !important;
}

/* Date input focus */
.daterangepicker-date-input:focus {
  border-color: #17a2b8 !important;
  box-shadow: 0 0 0 0.2rem rgba(23, 162, 184, 0.25) !important;
}

.daterangepicker-date-input.compare:focus {
  border-color: #ff9307 !important;
  box-shadow: 0 0 0 0.2rem rgba(255, 147, 7, 0.25) !important;
}
</style>
