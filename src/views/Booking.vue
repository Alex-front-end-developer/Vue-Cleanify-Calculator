<template>
  <div id="booking">
    <div class="date">
      <h2>Select the date of cleaning</h2>
      <VueSlickCarousel
          ref="c1"
          class="months"
          :asNavFor="$refs.c2"
          :focusOnSelect="true"
          :centerMode="true"
          :infinite="false"
          :variableWidth="true"
          @beforeChange="syncSliders"
      >
        <p
            v-for="month in 6"
            class="item"
        >
          {{$moment().add(month - 1, 'month').format('MMMM')}}
        </p>
      </VueSlickCarousel>
      <div class="dayOfWeek">
        <div class="item" v-for="day in date">
          <p class="day">{{day.day}}</p>
          <p class="sale">Up to ${{day.sale}} off</p>
        </div>
      </div>
      <VueSlickCarousel
          ref="c2"
          class="daysOfTheMonth"
          :focusOnSelect="true"
          :asNavFor="$refs.c1"
          :infinite="false"
          @beforeChange="syncSliders"
      >
        <div class="month" v-for="month in 6">
          <div v-for="spaceDays in $moment().add(month - 1, 'month').startOf('month').day()"></div>
          <div
              v-for="day in $moment().add(month - 1, 'month').daysInMonth()"
              class="button"
              @click="setSale($moment().add(month - 1, 'month').startOf('month').add(day - 1, 'days').day(), day, month - 1)"
              :class="{
                active: +day === +data.date.day && (+$moment().add(month - 1, 'month').format('MM') === +data.date.month),
                disabled: +day < +$moment().format('DD') && +$moment().add(month - 1, 'month').format('MM') <= +$moment().format('MM') && +$moment().add(month - 1, 'month').format('YYYY') <= +$moment().format('YYYY')
              }"
          >
            {{day}}
          </div>
        </div>
      </VueSlickCarousel>
    </div>
    <div class="time">
      <h2>Indicate the convenient time</h2>
      <v-select
          class="item button"
          label="text"
          label-option="text"
          :options="times"
          v-model="data.date.time"
          style2
          reverse
          v-if="times.length"
      ></v-select>
    </div>
    <div class="buttons">
      <a
           @click="$router.go(-1)"
           class="back"
       >
         < Back
       </a>
      <router-link
          :to="{name: 'Booking-2'}"
          class="button active"
          tag="button"
          :disabled="data.date.time.hidden || !$moment(data.date.year + '-' + data.date.month + '-' + data.date.day).isSameOrAfter($moment().format('YYYY-MM-DD'))"
      >
        Next
      </router-link>
    </div>
  </div>
</template>

<script>
import VueSlickCarousel from 'vue-slick-carousel'
import 'vue-slick-carousel/dist/vue-slick-carousel.css'
import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css'
import VSelect from "@/components/VSelect";
export default {
  name: "booking",
  components: {
    VueSlickCarousel,
    VSelect
  },
  data() {
    return {
      tempDay: null
    }
  },
  methods: {
    syncSliders(currentPosition, nextPosition) {
      this.$refs.c1.goTo(nextPosition)
      this.$refs.c2.goTo(nextPosition)
    },
    setSale(dayOfWeek, day, month) {
      let oldDayOfWeek = this.$moment('' + this.data.date.year + this.data.date.month + this.data.date.day, "YYYYMMDD").day()
      // if (this.data.date.day !== null) this.$store.commit('subtotal', this.subtotal + +this.date[oldDayOfWeek].sale)
      this.data.date.day = day
      this.data.date.month = this.$moment().add(month, 'month').format('MM')
      this.data.date.year = this.$moment().add(month, 'month').format('YYYY')
      // this.$store.commit('subtotal', this.subtotal - +this.date[dayOfWeek].sale)
      this.$store.dispatch('setCache')
    }
  },
  computed: {
    date() {
      return this.$store.state.date;
    },
    times() {
      return this.$store.state.times;
    },
    subtotal() {
      return this.$store.state.subtotal;
    },
    data() {
      return this.$store.state.dataToSend;
    },
  }
}
</script>

<style lang="scss">
  #booking {
    display: flex;
    flex-direction: column;
    padding-bottom: vwD(350);
    height: calc(100vh - #{vwD(87)});
    .date {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 0 auto;
      padding: 0 vwD(50);
      .months {
        width: 70%;
        margin-top: vwD(20);
        .slick-prev, .slick-next {
          width: vwD(8);
          height: vwD(13);
          background: url('~@/assets/img/icons/arrowRight.svg') no-repeat 0 0 / contain;
          transform: none;
          right: 40%;
          left: auto;
          top: 0;
          bottom: 0;
          margin: auto;
          z-index: 1;
          &.slick-disabled {
            opacity: .3;
          }
          &:before {
            display: none;
          }
        }
        .slick-prev {
          transform: rotate(180deg);
          left: 40%;
          right: auto;
        }
        .slick-track {
          display: flex;
          align-items: center;
        }
        .slick-slide {
          opacity: .4;
          transform: scale(.8);
          transition: all 0.5s;
          padding: 0 vwD(43);
          &.slick-center {
            opacity: 1;
            transform: none;
          }
        }
        .item {
          text-align: center;
          transition: all 0.5s;
          font-size: vwD(25);
          cursor: pointer;
        }
      }
      .dayOfWeek {
        display: grid;
        grid-template-columns: repeat(7, 1fr);
        grid-column-gap: vwD(17);
        width: 100%;
        margin-top: vwD(70);
        .item {
          display: flex;
          align-items: center;
          flex-direction: column;
          .day {
            opacity: .4;
          }
          .sale {
            opacity: .5;
            margin-top: vwD(14);
          }
        }
      }
      .daysOfTheMonth {
        width: 100%;
        margin-top: vwD(14);
        overflow: hidden;
        .month {
          display: grid !important;
          grid-template-columns: repeat(7, 1fr);
          grid-column-gap: vwD(17);
          grid-row-gap: vwD(22);
        }
      }
    }
    .time {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      h2 {
        margin-bottom: vwD(40);
      }
      .v-select {
        width: vwD(450);
      }
    }
  }
  @media screen and (max-width: $mobileOn) {
    #booking {
      .date {
        padding: 0;
        .months {
          width: 100%;
          padding: vwM(20) vwM(15);
          border: 1px solid $primary;
          border-bottom: 0;
          border-radius: vwM(16) vwM(16) 0 0;
          .item {
            font-size: vwM(22);
          }
          .slick-slide {
            opacity: 0;
          }
          .slick-prev {
            width: vwM(15);
            height: vwM(15);
            left: 22%;
          }
          .slick-next {
            width: vwM(15);
            height: vwM(15);
            right: 22%;
          }
        }
        .dayOfWeek {
          margin-top: 0;
          padding: 0 vwM(15) vwM(15) vwM(15);
          border: 1px solid $primary;
          border-top: 0;
          border-bottom: 0;
          .item {
            .day {
              font-size: vwM(13);
            }
            .sale {
              font-size: vwM(10);
              text-align: center;
              margin-top: vwM(10);
            }
          }
        }
        .daysOfTheMonth {
          margin-top: 0;
          padding-bottom: vwM(20);
          border: 1px solid $primary;
          border-radius: 0 0 vwM(16) vwM(16);
          border-top: 0;
          .month {
            grid-column-gap: vwM(13);
            grid-row-gap: vwM(10);
            padding: 0 vwM(15);
          }
          .button {
            padding: 0;
            border-radius: vwM(5);
            width: vwM(25);
            height: vwM(25);
            font-size: vwM(12);
          }
        }
      }
      .time {
        h2 {
          margin-bottom: vwM(20);
        }
        .v-select {
          width: 100%;
        }
      }
    }
  }
</style>
