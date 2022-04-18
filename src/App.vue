<template>
    <div class="lucky-wheel">
      <div class="pointer-container">
        start
        <div class="pointer" ref="pointer" id="pointer" :style="{'transform':rotate_deg,'transition': prize_transition}"
          @click="rotateHandler(num)">
          <span></span>
        </div>
      </div>
  
      <div class="container">
        <div v-for="(item) in prizes" :key="item.name" ref="item" :class="itemClass">
          <div class="item-content">

            <div>{{item.name}}
              <span class="count">{{item.count}}</span>
            </div>
          </div>
        </div>
      </div>

      <transition name="slide-fade">
        <div class="prize" v-if="isShow==isClicked">
          <div class="prize-container">
            <div class="prize-title">
              恭喜您抽中了 : {{prize_name}} 
            </div>
          </div>
        </div>
      </transition>
    </div>
</template>

<style>

</style>

<script>


import axios from "axios";
export default {
   data (){

  return {
      prizes: [],
      prizes_2017: [],
      prize_name: '',
      prize_icon: '',
      prize_rotate: [],
      prize_transition: '',
      each_deg: 0,
      rotate_deg: 0,
      start_deg: 0,
      current_deg: 0,
      index: 0,
      current_year: 2017,
      duration: 3000,
      time_remaining: 20,
      num: 0,
      numbers: [],//紀錄還有獎品的編號
      isClicked: false,//轉動中禁止觸發  
      isShow: true,
  }
     
    },
    mounted() {
      let vm = this
      vm.initPrize()
    },

    computed: {
      // 判斷轉盤 class
      itemClass() {
        let vm = this
        return vm.current_year === 2017 ? 'item item-skew' : 'item item-skew-large'
      },
  
    },
    methods: {
      prizeActive() {
       
        let vm = this
        setTimeout(() => {
          vm.$refs.item[vm.index].classList.value = `${vm.itemClass} active`
        }, vm.duration);
        console.log('item active')
      },
    
      initPrize() {
        let vm = this
        axios.get('./prize20.json')
          .then(function (response) {
            vm.prizes_2017 = JSON.parse(response.request.responseText)
            console.log(vm.prizes_2017);
            vm.num = vm.prizes_2017.length
            vm.degree(vm.num)
            vm.prizes = vm.prizes_2017
            vm.numberArray()
          })
          .catch(function (error) {
            console.log(error);
          });
      },

      degree(num) {
  
        let vm = this
        for (let i = 1; i <= num; i++) {
          let deg = 360 / num
          vm.each_deg = deg
          let eachDeg
          eachDeg = i * deg
          vm.prize_rotate.push(eachDeg)
        }
      },
      numberArray() {
        let vm = this
        vm.numbers = vm.prizes.map((prize, index) => {
          return index
        })
      },
      rotateHandler(num) {
        let vm = this

        vm.prizes.filter((prize, index) => {
          let filterArray
          if (prize.count <= 0) {
             filterArray = vm.numbers.filter((num) => {
              return num !== index
            })
            vm.numbers = filterArray
          }
        })

        if (vm.time_remaining > 0) {
          vm.$refs.item[vm.index].classList.value = vm.itemClass
 
          vm.prize_draw(num)
        } else if (vm.time_remaining <= 0) {
          alert("是否重製獎品數量")

        }
      },
      prize_draw() {
     
        let vm = this
        if (vm.isClicked) return
        vm.isShow = vm.isClicked

        vm.$refs.item[vm.index].classList.value = vm.itemClass


        vm.index = vm.numbers[Math.floor(Math.random() * vm.numbers.length)]
        console.log('1.剩餘牌號', vm.numbers)

   
        let circle = 3
        let degree
 
        degree = vm.start_deg + circle * 360 + vm.prize_rotate[vm.index] - vm.start_deg % 360 + 45

        vm.start_deg = degree
    
        vm.rotate_deg = `rotate(${degree}deg)`

        vm.prize_transition = `all ${vm.duration / 1000}s cubic-bezier(0.42, 0, 0.2, 0.91)`
        vm.time_remaining--
        vm.isClicked = true


        let remainder = vm.start_deg % 360
        if (remainder <= 0) {
 
          vm.current_deg = remainder + 360 

        } else if (remainder > 0) {
          vm.current_deg = remainder
        }
        console.log('2.執行旋轉', degree, 'index', vm.index)


        let prize = vm.prizes[vm.index]
        vm.prize_name = prize.name
     
        vm.prizeActive()
        setTimeout(() => {
          prize.count--
          console.log('3.旋轉角度:', vm.current_deg, '獎品:', prize.name, '剩餘數量:', prize.count, ' index', vm.index)
        }, vm.duration);


        setTimeout(() => {
          if (vm.isClicked === true) {
            vm.isClicked = false
          }
        }, vm.duration);
      }
    },
}
</script>

<style>
*,
*::before,
*::after {
  -webkit-box-sizing: border-box;
          box-sizing: border-box;
  margin: 0;
  padding: 0;
}

[v-cloak] {
  display: none !important;
}

body {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  height: 100vh;
  overflow: hidden;
  position: relative;
  background-color: #edf5ff;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
}

.status-container {
  position: relative;
  left: 100px;
}

.toggle {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  background-position: center;
  background-size: 100% 100%;
  background-repeat: no-repeat;
  width: 100px;
  height: 120px;
  position: absolute;
  left: calc;
  top: -3%;
  -webkit-transition: all 0.3s ease-in;
  transition: all 0.3s ease-in;
  -webkit-filter: drop-shadow(-3px 5px 0px #ffc7c9);
          filter: drop-shadow(-3px 5px 0px #ffc7c9);
  cursor: pointer;
}

.toggle:hover {
  -webkit-transform: rotate(360deg) scale(0.8);
          transform: rotate(360deg) scale(0.8);
}

.game-status {
  position: absolute;
  left: 2%;
  top: 4%;
  font-size: 2rem;
  color: #ffc7c9;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: end;
      -ms-flex-pack: end;
          justify-content: flex-end;
  -webkit-box-align: stretch;
      -ms-flex-align: stretch;
          align-items: stretch;
  margin: 0 auto;
  -webkit-box-orient: vertical;
  -webkit-box-direction: reverse;
      -ms-flex-direction: column-reverse;
          flex-direction: column-reverse;
  text-transform: uppercase;
  width: 250px;
  height: 600px;
}

button:focus {
  outline: 0;
}

.times {
  font-size: 2.5rem;
  line-height: 0.9;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  -webkit-box-pack: start;
      -ms-flex-pack: start;
          justify-content: flex-start;
  margin: 5px 0px;
  position: relative;
  color: #1f1172;
}

.times::after {
  position: absolute;
  content: "x";
  font-size: 2.3rem;
  left: 150px;
  top: 3px;
  color: #ffc7c9;
}

.times > span {
  text-align: right;
  color: #1f1172;
  margin-left: 40px;
}

.buttons {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-orient: horizontal;
  -webkit-box-direction: normal;
      -ms-flex-direction: row;
          flex-direction: row;
  -ms-flex-pack: distribute;
      justify-content: space-around;
}

.btn {
  cursor: pointer;
  width: 100px;
  padding: 5px 10px;
  margin: 10px 5px;
  font-size: 1rem;
  -webkit-transition: all 0.3s;
  transition: all 0.3s;
}

.btn.btn-primary {
  border: 3px solid #1f1172;
  color: #ffc7c9;
  background-color: #ffc7c9;
}

.btn.active {
  border: 3px solid #e94e4e;
  color: #e94e4e;
  background-color: #1f1172;
}

.btn.btn-secondary {
  border: 3px solid #1f1172;
  color: #ffc7c9;
  background-color: #ffc7c9;
}

.btn.btn-secondary:hover {
  border: 3px solid #ffc7c9;
  color: #ffc7c9;
  background-color: #1f1172;
}

.lucky-wheel {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  width: 550px;
  height: 550px;
  border-radius: 550px;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  z-index: 3;
}

.lucky-wheel .container {
  display: block;
  width: 520px;
  height: 520px;
  border-radius: 520px;
  overflow: hidden;
  position: relative;
}

.pointer-container {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  width: 90px;
  height: 90px;
  -ms-flex-negative: 0;
      flex-shrink: 0;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  background-color: #fff;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  z-index: 900;
  position: absolute;
  color: #e94e4e;
  font-size: 2rem;
}

.pointer-container .pointer {
  width: 128px;
  height: 208px;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  border: 3px solid transparent;
  position: absolute;
  bottom: 39.7px;
  -webkit-transform-origin: 64px 204px;
          transform-origin: 64px 204px;
  cursor: pointer;
  z-index: 900;
}

.pointer-container .pointer span {
  display: block;
  width: 0;
  height: 0;
  border-style: solid;
  border-color: transparent;
  border-width: 108px 20px 80px 20px;
  border-top-color: #797979;
  margin-top: -140px;
}

.item {
  position: absolute;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  width: 50%;
  height: 50%;
  border: 1px solid #ffffff;
  top: 0;
  right: 0;
  -webkit-transform-origin: 0% 100%;
          transform-origin: 0% 100%;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
}

.item-skew:nth-child(1) {
  -webkit-transform: rotate(90deg) skewY(0deg);
          transform: rotate(90deg) skewY(0deg);
}

.item-skew:nth-child(2) {
  -webkit-transform: rotate(180deg) skewY(0deg);
          transform: rotate(180deg) skewY(0deg);
}

.item-skew:nth-child(3) {
  -webkit-transform: rotate(270deg) skewY(0deg);
          transform: rotate(270deg) skewY(0deg);
}

.item-skew:nth-child(4) {
  -webkit-transform: rotate(360deg) skewY(0deg);
          transform: rotate(360deg) skewY(0deg);
}

.item-content {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  width: 120px;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
      -ms-flex-direction: column;
          flex-direction: column;
  font-size: 2rem;
  -webkit-transform: rotate(45deg) translate(-95px, 0px);
          transform: rotate(45deg) translate(-95px, 0px);
  position: absolute;
  right: 0;
  bottom: 0;
}

.item-content .count {
  position: absolute;
  left: 28px;
  top: 112px;
  font-size: 1.2rem;
  text-align: center;
  width: 45px;
  line-height: 25px;
  display: block;
}

.item-content > i {
  font-size: 4rem;
}

.item {
  background-color: #ffc7c9;
  color: #000;
}

.item .count {
  background-color: #ffc7c9;
}

.item.active {
  background-color: #e94e4e;
  -webkit-transition: 0.2s ease-in;
  transition: 0.2s ease-in;
}

.item.active .item-content {
  color: white;
  -webkit-transition: 0.2s ease-in;
  transition: 0.2s ease-in;
}

.prize {
  position: absolute;
  width: 100%;
  height: 140px;
  background-color: #ffffff;
  overflow: hidden;
  top: 0;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  padding-top: 2%;
  z-index: 100000;
}

.prize .prize-container {
  width: 1280px;
  position: relative;
  z-index: 100000;
}

.prize .prize-title {
  font-size: 36px;
  color: black;
  z-index: 1000;
  text-align: center;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 0.2s;
          animation-delay: 0.2s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 0.4s;
          animation-delay: 0.4s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 0.6s;
          animation-delay: 0.6s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 0.8s;
          animation-delay: 0.8s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 1s;
          animation-delay: 1s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 1.2s;
          animation-delay: 1.2s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 1.4s;
          animation-delay: 1.4s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 1.6s;
          animation-delay: 1.6s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.prize-background i:nth-child(odd) {
  -webkit-animation: move 3s alternate infinite ease-in-out;
          animation: move 3s alternate infinite ease-in-out;
  -webkit-animation-delay: 1.8s;
          animation-delay: 1.8s;
}

.prize-background i:nth-child(even) {
  animation: move 2s alternate-reverse infinite ease-in-out;
}

.slide-fade-enter-active {
  -webkit-transition: all 1s ease;
  transition: all 1s ease;
  -webkit-transition-delay: 2s s;
          transition-delay: 2s s;
}

.slide-fade-leave-active {
  -webkit-transition: all 0.4s cubic-bezier(1, 0.5, 0.8, 1);
  transition: all 0.4s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter,
.slide-fade-leave-to {
  -webkit-transform: translateX(10px);
          transform: translateX(10px);
  opacity: 0;
}
/*# sourceMappingURL=style.css.map */
</style>