<template>
  <div class="v-select" :class="{open: visible, style2: style2, reverse: reverse, mobile: mobileStyle, button: mobileStyle}" @click="visible = !visible" v-click-outside="clickOutside">
    <div class="text">
      <div class="title">
        <p>{{ value[label] }}</p>
<!--        <span v-if="value.duration">~ {{ +value.duration + details }} hour</span>-->
        <span v-if="+value.sale">save$ {{value.sale }}.00</span>
      </div>
      <img src="@/assets/img/icons/arrowDown.svg" alt=""/>
    </div>
    <div class="v-list-dropdown" :class="{calendar: options[0].type === 'date', addons: options[0].type === 'addons', zip: options[0].text === 'Zip code'}" v-if="options[0]">
      <div class="zip" v-if="options[0].text === 'Zip code'">
        <input type="text"
               maxlength="5"
               v-mask="'#####'"
               v-model="data.zip"
               @click.stop=""
               :readonly="data.address.status"
        >
      </div>
      <div class="date" v-else-if="options[0].type === 'date'" @click.stop>
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
          >
            {{$moment().add(month - 1, 'month').format('MMMM')}}
          </p>
        </VueSlickCarousel>
        <div class="dayOfWeek">
          <div class="item" v-for="day in date">
            <p class="day">{{day.day}}</p>
            <p class="sale">${{day.sale}} off</p>
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
                disabled: $moment().add(month - 1, 'month').set('date', day - 2).isBefore($moment())
              }"
            >
              {{day}}
            </div>
          </div>
        </VueSlickCarousel>
        <p style="text-align: right;">{{$moment(data.date.year + '-' + data.date.month + '-' + data.date.day).format('ddd. Do MMMM')}}</p>
      </div>
      <div class="addons" v-else-if="options[0].type === 'addons'" @click.stop>
        <div
            class="item button addon"
            :class="{
              active: data.addons.map(addon => addon.text).includes(item.text) || item.addons_included.includes(data.selected.typecleaning.text),
              disabled: item.addons_included.includes(data.selected.typecleaning.text)
            }"
            v-for="item in addons"
            :key="item.text"
            @click="selectedAddons(item)"
        >
          <div class="icon">
            <img :src="item.icon" alt=""/>
          </div>
          <p class="name">{{ item.text }}</p>
          <p class="price">${{ item.price }}</p>
        </div>
      </div>
      <ul ref="select" v-else>
        <li
            v-for="(option, i) in options"
            @click="selected(option, i)"
            :class="{active: value[labelOption] === option[labelOption]}"
        >
          {{ option[labelOption] }}
<!--          <span v-if="option.duration">~ {{ +option.duration + details }} hour</span>-->
          <span v-if="+option.sale">save ${{option.sale}}.00</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import {mask} from "vue-the-mask";
import VueSlickCarousel from 'vue-slick-carousel'
import 'vue-slick-carousel/dist/vue-slick-carousel.css'
import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css'

export default {
  name: 'v-select',
  components: {
    VueSlickCarousel
  },
  directives: {
    mask
  },
  props: {
    options: {
      type: Array,
      required: true
    },
    labelOption: {
      type: String,
      default: 'text'
    },
    label: {
      type: String,
      default: 'text'
    },
    value: {
      type: Object
    },
    details: {},
    style2: {
      type: Boolean,
      default: false
    },
    reverse: {
    }
  },
  data() {
    return {
      visible: false
    }
  },
  computed: {
    mobileStyle() {
      return window.innerWidth < 1023
    },
    data() {
      return this.$store.state.dataToSend
    },
    date() {
      return this.$store.state.date;
    },
    addons() {
      return this.$store.state.addons
    },
  },
  methods: {
    clickOutside() {
      this.visible = false
    },
    selected(option, i) {
      option.index = i
      this.$emit('input', option)
      this.$store.dispatch('setCache')
    },
/*    detailsValue(item) {
      return item[this.details.match(/[{].*[}]/)[0].replace(/\W/gi, '')]
    },*/
    syncSliders(currentPosition, nextPosition) {
      this.$refs.c1.goTo(nextPosition)
      this.$refs.c2.goTo(nextPosition)
    },
    setSale(dayOfWeek, day, month) {
      let oldDayOfWeek = this.$moment('' + this.data.date.year + this.data.date.month + this.data.date.day, "YYYYMMDD").day()
      // if (this.data.date.day !== null) this.$store.commit('subtotal', this.subtotal + +this.date[oldDayOfWeek].sale)
      this.data.date.day = String(day).length < 2 ? '0' + day : day
      this.data.date.month = this.$moment().add(month, 'month').format('MM')
      this.data.date.year = this.$moment().add(month, 'month').format('YYYY')
      // this.$store.commit('subtotal', this.subtotal - +this.date[dayOfWeek].sale)
      this.visible = false
      this.$store.dispatch('setCache')
    },
    selectedAddons(item) {
      if (this.data.addons.map(addon => addon.text).includes(item.text)) {
        this.$store.commit('dataToSend', {key: 'addons', payload: this.data.addons.filter(addon => addon.text !== item.text)})
      } else {
        this.$store.commit('dataToSend', {key: 'addons', payload: [...this.data.addons, item]})
      }
      this.$store.dispatch('setCache')
    }
  },
  /*created() {
    !this.value && this.$emit('update:value', this.options[0])
  },*/
}
</script>

<style lang="scss">
.v-select {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;

  .text {
    display: flex;
    align-items: center;
    justify-content: center;

    .title {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      //font-weight: 700;
      position: relative;
      white-space: nowrap;
      p {
        font-size: vwD(20);
      }

      span {
        font-size: vwD(13);
        font-weight: 600;
        position: absolute;
        top: 110%;
        left: 0;
        right: 0;
        display: flex;
        align-items: center;
        justify-content: center;
      }
    }

    img {
      margin-left: vwD(15);
      transition: all 0.2s;
      width: vwD(15);
    }
  }

  .v-list-dropdown {
    position: absolute;
    left: 0;
    min-width: 100%;
    top: 100%;
    border-radius: 0 0 vwD(20) vwD(20);
    background: #FBFBFB;
    padding: 1px 0;
    opacity: 0;
    visibility: hidden;
    overflow: hidden;
    z-index: 100;
    box-shadow: 0 vwD(15) vwD(30) rgba(0, 0, 0, 0.1);

    &.calendar {
      border-radius: vwD(16);
    }

    &.addons {
      left: auto;
      right: 0;
    }

    .zip {
      padding: vwD(15) vwD(25);
      input {
        border-width: 0;
        border-bottom: 1px solid transparentize(#161F32, .9);;
        padding-bottom: vwD(5);
        background: none;
        color: transparentize($color, .3);
      }
    }

    ul {
      list-style: none;
      max-height: vwD(320);
      overflow: auto;
      &::-webkit-scrollbar {
        width: 2px;
        border-radius : 20px;
      }

      &::-webkit-scrollbar-track {
        background: none;
      }

      &::-webkit-scrollbar-thumb {
        background: $secondary;
        border-radius : 20px;
        width: 2px;
      }

      &::-webkit-scrollbar-thumb:window-inactive {
        background: $secondary;
        border-radius : 20px;
        width: 2px;
      }
      li {
        padding: 0 vwD(25);
        width: 100%;
        height: vwD(40);
        transition: all 0.2s;
        display: flex;
        align-items: center;
        justify-content: space-between;
        white-space: nowrap;
        flex-grow: 1;
        span {
          opacity: .5;
          font-size: vwD(12);
          margin-left: vwD(10);
        }
        &:hover {
          background: transparentize($primary, .85);
        }

        &.active {
          background: transparentize($primary, .70);
        }
      }
    }

    .date {
      width: vwD(300);
      border: 1px solid $primary;
      border-radius: vwD(16);
      padding: vwD(20);
      cursor: auto;
      .months {
        width: 100%;
        .slick-prev, .slick-next {
          width: vwD(15);
          height: vwD(15);
          background: url('~@/assets/img/icons/arrowRight.svg') no-repeat 0 0 / contain;
          transform: none;
          right: 10%;
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
          left: 10%;
          right: auto;
        }
        .slick-track {
          display: flex;
          align-items: center;
        }
        .slick-slide {
          opacity: 0;
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
        grid-column-gap: vwD(9);
        width: 100%;
        margin-top: vwD(15);
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
            text-align: center;
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
          grid-column-gap: vwD(13);
          grid-row-gap: vwD(10);
          .button {
            width: vwD(25);
            height: vwD(25);
            border-radius: vwD(5);
            font-size: vwD(12);
          }
        }
      }
    }
    .addons {
      width: vwD(300);
      padding: vwD(10);
      .item:not(:last-of-type) {
        margin-bottom: vwD(10);
      }
      .item {
        padding: 0 vwD(18) 0 vwD(10);
        height: vwD(40);
        border-radius: vwD(10);
        .icon {
          margin-right: vwD(15);
        }
        .price {
          margin-left: auto;
        }
        &.addon {
          &.active {
            background: transparentize($primary, .70);
            color: $color;
            .name, .price {
              opacity: 1;
            }
          }
        }
      }
    }
  }

  &.open {
    .text {
      img {
        transform: rotate(180deg);
      }
    }

    .v-list-dropdown {
      opacity: 1;
      visibility: visible;
    }
  }

  &.style2, &.mobile {
    padding: 0 vwD(25) !important;
    &:hover {
      background: none !important;
    }
    .text {
      justify-content: space-between;
      width: 100%;
      .title {
        opacity: .7;
        font-weight: 500;
        font-size: vwD(16);
        flex-direction: row;
        justify-content: space-between;
        flex-grow: 1;
        span {
          position: static;
          opacity: .5;
        }
      }
    }
    .v-list-dropdown {
      border: 1px solid $secondary;
      border-top-color: $primary;
      left: -1px;
      width: calc(100% + 2px);
      background: #fff;
    }
    &.open {
      border-radius: vwD(16) vwD(16) 0 0 !important;
      border-color: $secondary !important;
    }
    &.reverse {
      .v-list-dropdown {
        top: auto;
        bottom: 100%;
        border-radius: vwD(20) vwD(20) 0 0;
        box-shadow: 0 (-vwD(15)) vwD(30) rgba(0, 0, 0, 0.1);
        border: 1px solid $secondary;
        border-bottom-color: $primary;
      }
      &.open {
        border-radius: 0 0 vwD(16) vwD(16) !important;
      }
    }
  }
}
@media screen and (max-width: $mobileOn) {
  .v-select {
    &.style2, &.mobile {
      padding: 0 vwM(25) !important;
      &:hover {
        background: #fff !important;
      }
      .text {
        .title {
          p {
            font-size: vwM(16);
          }
          span {
            font-size: vwM(12);
            text-align: center;
          }
        }
        img {
          width: vwM(13);
          margin-left: vwM(10);
        }
      }
      .v-list-dropdown {
        border-radius: 0 0 vwM(16) vwM(16);
        padding: vwM(4) 0;
        border-width: vwM(1);
        left: -vwM(1);
        width: calc(100% + #{vwM(2)});
        ul {
          max-height: vwM(210);
          li {
            height: auto;
            padding: vwM(8) vwM(25);
            white-space: normal;
            span {
              font-size: vwM(12);
              text-align: center;
              white-space: nowrap;
            }
          }
        }
        &.calendar {
          position: fixed;
          background: transparentize($secondary, .5);
          width: 100%;
          height: 100vh;
          border-radius: 0;
          border: 0;
          top: 0;
          left: 0;
          display: flex;
          align-items: center;
          justify-content: center;
          .date {
            background: #ffffff;
            border-radius: vwM(16);
            width: 95%;
            padding: vwM(15) 0;
            &>p {
              padding-right: vwM(15);
            }
            .months {
              width: 100%;
              padding: 0 vwM(15);
              //border: 1px solid $primary;
              border-bottom: 0;
              border-radius: vwM(16) vwM(16) 0 0;
              margin-bottom: vwM(15);
              .item {
                font-size: vwM(22);
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
              padding: 0 vwM(15);
              margin-bottom: vwM(10);
              //border: 1px solid $primary;
              border-top: 0;
              border-bottom: 0;
              .item {
                border: none !important;
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
              //border: 1px solid $primary;
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
        }
        .addons {
          width: 100%;
          padding: vwM(10);
          .item {
            padding: 0 vwM(15);
            border-radius: vwM(16);
            .icon {
              margin-right: vwM(10);
            }
          }
        }
        &.zip {
          padding: vwM(10);
          input {
            width: 100%;
          }
        }
      }
      &.open {
        border-radius: vwM(16) vwM(16) 0 0 !important;
      }
      &.reverse {
        .v-list-dropdown {
          border-radius: vwM(16) vwM(16) 0 0;
        }
        &.open {
          border-radius: 0 0 vwM(16) vwM(16) !important;
        }
      }
    }
    &.open {
      z-index: 10;
    }
  }
}
</style>
