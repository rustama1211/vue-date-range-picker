<template>
  <div class="daterangepicker-calendar">
    <div class="d-flex align-items-center">
      <div class="p-1" :class="calendarIndex == 1 ? '' : 'invisible'">
        <button type="button" class="btn btn-sm btn-light" @mousedown.prevent @click="goToPrevMonth">
          <font-awesome-icon icon="caret-left" fixed-width />
        </button>
      </div>
      <div class="p-1 col text-center">
        {{ displayMonth.format('MMMM YYYY') }}
      </div>
      <div class="p-1" :class="calendarIndex == calendarCount ? '' : 'invisible'">
        <button type="button" class="btn btn-sm btn-light" @mousedown.prevent @click="goToNextMonth">
          <font-awesome-icon icon="caret-right" fixed-width />
        </button>
      </div>
    </div>
    <div class="d-flex justify-content-between text-center daterangepicker-calendar-row">
      <div v-for="day in daysOfFirstWeek" :key="day.format('D')" class="col-day">{{ day.format('ddd') }}</div>
    </div>
    
    <div class="d-flex flex-wrap justify-content-between text-center daterangepicker-calendar-row">
      <div v-for="day in days" :key="day.format('M-D')" class="col-day"
        :class="dayClass(day)" @mouseover="dayMouseOver(day)" @mousedown.prevent @click="dayClick(day)">{{ day.format('D') }}</div>
    </div>
  </div>
</template>

<script>
import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
import fontawesome from '@fortawesome/fontawesome'
import faCaretLeft from '@fortawesome/fontawesome-free-solid/faCaretLeft'
import faCaretRight from '@fortawesome/fontawesome-free-solid/faCaretRight'
import moment from 'moment'

fontawesome.library.add(faCaretLeft, faCaretRight)

export default {
  props: ['calendarIndex', 'calendarCount', 'month', 'startDate', 'endDate', 'compare', 'startDateCompare', 'endDateCompare', 'step'],
  data: () => {
    return { last_step: null}
  },
  computed: {
    displayMonth: function() {
      return moment(this.month).add(this.calendarIndex - 1, 'month')
    },
    days: function() {
      let startDay = moment(this.displayMonth).startOf('isoWeek').subtract(1,'day');
      let endDay = moment(this.displayMonth).endOf('month').endOf('isoWeek').startOf('day').add(1,'day');
      let nDays = moment.duration(endDay.diff(startDay)).asDays()

      let days = []
      let day = 0
      while (day < nDays) {
        days.push(moment(startDay).add(day, 'day'))
        day++
      }
      return days
    },
    daysOfFirstWeek: function() {
      return this.days.slice(0, 7)
    }
  },
  methods: {
    dayClass: function(day) {
      let classes = []
      
      // Hide days overflowing
      if (!day.isBetween(this.displayMonth, moment(this.displayMonth).endOf('month'), 'days', '[]')) {
        classes.push('invisible')
      }
      // Class for days between startDate & endDate or is startDate (in case of startDate after endDate)
      if (day.isBetween(this.startDate, this.endDate, 'days', '[]') || day.isSame(this.startDate)) {
        classes.push('daterangepicker-range')
      } 
      
      // Class for days between startDateCompare & endDateCompare or is startDateCompare (in case of startDateCompare after endDateCompare)
      if (this.compare && (day.isBetween(this.startDateCompare, this.endDateCompare, 'days', '[]') || day.isSame(this.startDateCompare))) {
        classes.push('daterangepicker-range-compare')
      }
      // Class for cursor if the step is selecting something
      if (this.step != null) {
        classes.push('daterangepicker-cursor-pointer')
      }

      if ( ( this.step == 'selectEndDate' && day.isBefore(this.startDate)) || (this.step == 'selectEndDateCompare' && day.isBefore(this.startDateCompare)) ) {
        classes.push('date-disabled');
      }

      return classes.join(' ')
    },
    dayMouseOver: function(day) {
     /* if (this.step != null) {
        this.selectDate(day)
      }*/
    },
    dayClick: function(day) {
     
      if (this.step != null) {
       
        if(this.step == 'selectEndDate' && day.isBefore(this.startDate)) {
          
          return;
        }
        if(this.step == 'selectEndDateCompare' && day.isBefore(this.startDateCompare)) {
          return;
        }
        
        this.selectDate(day);
        this.nextStep()
      }else
      {
        
        if(this.step == 'selectEndDate' && day.isBefore(this.startDate)) {
          return;
        }
        if(this.step == 'selectDndDateCompare' && day.isBefore(this.startDateCompare)) {
          return;
        }
        this.nextStep();
        this.selectDate(day);
        this.nextStep();
        

      }

    },
    goToPrevMonth: function() {
      this.$emit('goToPrevMonth')
    },
    goToNextMonth: function() {
      this.$emit('goToNextMonth')
    },
    selectDate: function(date) {
      this.$emit('selectDate', date)
      //this.$forceUpdate()
    },
    nextStep: function() {
      this.$emit('nextStep')
    }
  },
  watch: {
    step: function(val) {
      if(val) {
        this.last_step = val;
      }
    }
  },
  filters: {},
  components: { FontAwesomeIcon }
}
</script>

<style>
div.col-day:not(.date-disabled):hover {
  background-color:#fc3;
}
.daterangepicker-calendar-row {
  font-size: 14px;
  /*max-width: 230px;*/
}

.date-disabled {
  background-color: #ddd;
  color: #ffffff;
  cursor: not-allowed !important;
}

.col-day {
  width: 14.28%;
  padding: 0.5rem 0;
  white-space: nowrap;
  cursor: default;
}

.daterangepicker-range {
  background-color: #17a2b8 !important;
  color: #ffffff;
}

.daterangepicker-range-compare {
  background-color: #ff9307;
  color: #ffffff;
}

.daterangepicker-range.daterangepicker-range-compare {
  background: linear-gradient(0, #ff9307 50%, #17a2b8 50%);
}

.daterangepicker-cursor-pointer {
  cursor: pointer;
}
</style>
