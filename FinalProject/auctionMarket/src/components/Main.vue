<template>
<div id = "default">
  <my-header></my-header>
  <main>
    <h2>■ 현재시간 : <span id="nowTimes"></span></h2>
    <div v-for="product in sortedProducts" :key="product.id">
      <div class="row">
        <div class="col-md-5 col-md-offset-0">
          <figure>
            <img class="product" v-bind:src="product.image" width="250" height="250">
          </figure>
        </div>
        <div class="col-md-6 col-md-offset-0 description">
            <h3><b>{{product.title}}</b></h3>
          <p v-html="product.description"></p>
          <p class="price">
            현재 입찰가 :{{product.price | formatPrice}}
          </p>
          <p class = "limit">
            경매 종료 : {{product.limitDate | formatDate}} {{product.limitTime | formatTime}}
          </p>
          <p>
            남은 시간 : {{product.remaindTime | formatTimeCount}}
          </p>
          <router-link tag="h1"
              :to="{name: 'Id', params: {id: product.id, price: product.price}}">
          <button class="btn btn-primary btn-lg"
            v-if="(product.remaindTime > 0)">입찰하기</button>
          <button disabled="true" class="btn btn-primary btn-lg"
            v-else>입찰하기</button>
            </router-link>
          <span class="buyPossible-message"
                v-if="product.remaindTime < 0">
              시간 초과!
          </span>
          <span class="buyPossible-message"
                v-else>지금 구매하세요!
          </span>
        </div>
      </div><!-- end of row -->
      <hr />
    </div><!-- end of v-for -->
  </main>
</div>
</template>

<script>
import MyHeader from './Header.vue';
export default {
  name: 'imain',
  data() {
    return {
      products: [],
      remaindTimerId: null,
      executionCount: 0,
    }
  },

  components: { MyHeader},
  methods: {
    //실시간 남은 시간을 product에 remaindTime에 값을 입력 함
    reloadRemaindTime(product){ 
      this.remaindTimerId = setTimeout(()=>{
        const now = new Date();
        const tempProduct = product;
          var year = now.getFullYear();
          var month= now.getMonth() + 1;
          if(month < 10) {
            month = "0" + month;
          }
          var day = now.getDate();
          if(day < 10) {
            day = "0" + day;
          }
          var hour = now.getHours();
          if(hour < 10) {
            hour = "0" + hour;
          }
          var min = now.getMinutes();
          if(min < 10) {
            min = "0" + min;
          }
          var sec = now.getSeconds();
          if(sec < 10) {
            sec = "0" + sec;
          }
          var realtime = year + "-" + month + "-" + day + " " + hour + ":" + min + ":" + sec;

          let productDate = product.limitDate.substr(0,4) + "-" + product.limitDate.substr(4,2) + "-" + product.limitDate.substr(6,2);
          let productTime = product.limitTime.substr(0,2) + ":" + product.limitTime.substr(2,2) + ":" + product.limitTime.substr(4,2);

          var productLimitTime = productDate + " " + productTime
          const startTime = new Date(realtime);
          const endTime = new Date(productLimitTime);
          var diffTime = (endTime.getTime() - startTime.getTime()) / (1000);

          const p = product;
          p.remaindTime = `${diffTime}`;         
        
        //  clearTimeout을 하게되면 product의 length 의 값이 아닌 번씩 반복이 되서 삭제 함 (12/05일 수정)
        //clearTimeout(this.remaindTimerId);
        this.reloadRemaindTime(tempProduct);
        
      }, 1000);
    },
  },
  computed: {
    sortedProducts() {
      if (this.products.length > 0) {
        let productsArray = this.products.slice(0);
        function compare(a, b) {
          if (a.title.toLowerCase() < b.title.toLowerCase())
            return -1;
          if (a.title.toLowerCase() > b.title.toLowerCase())
            return 1;
          return 0;
        }
        return productsArray.sort(compare);
      }
    },
    //첫번째 login.vue에서 realTimer() 함수가 있어 삭제함 (2022-12-07 수정)
  //   realTimer() {
  //     var nowDate = new Date();
  //     var year = nowDate.getFullYear();
  //     var month= nowDate.getMonth() + 1;
  //     var date = nowDate.getDate();
  //     var hour = nowDate.getHours();
  //     var min = nowDate.getMinutes();
  //     var sec = nowDate.getSeconds();
  //     if(month < 10) {
  //           month = "0" + month;
  //         }
  //         var date = now.getDate();
  //         if(day < 10) {
  //           day = "0" + day;
  //         }
  //         var hour = now.getHours();
  //         if(hour < 10) {
  //           hour = "0" + hour;
  //         }
  //         var min = now.getMinutes();
  //         if(min < 10) {
  //           min = "0" + min;
  //         }
  //         var sec = now.getSeconds();
  //         if(sec < 10) {
  //           sec = "0" + sec;
  //         }
  //     document.getElementById("nowTimes").innerHTML = 
  //       year + "-" + addzero(month) + "-" + addzero(date) + "&nbsp;" + hour + ":" + addzero(min) + ":" + addzero(sec);
  // },
  
  },
  filters: {
    formatPrice(price) {
      if (!parseInt(price)) {
        return '';
      }
      if (price > 99999) {
        var priceString = (price / 100).toFixed(2);
        var priceArray = priceString.split("").reverse();
        var index = 3;
        while (priceArray.length > index + 3) {
          priceArray.splice(index+3, 0, ',');
          index += 4;
        }
        return '$' + priceArray.reverse().join('');
      } else {
        return '$' + (price / 100).toFixed(2);
      }
    },
    formatDate(limitDate){
      var date = "";
      var strLimitDate = limitDate;
      date += strLimitDate.substr(0,4) + "-";
      date += strLimitDate.substr(4,2) + "-";
      date += strLimitDate.substr(6,2);
      return date;
    },
    formatTime(limitTime) {
      var time = "";
      var strLimitTime = limitTime;
      time += strLimitTime.substr(0,2) + ":";
      time += strLimitTime.substr(2,2) + ":";
      time += strLimitTime.substr(4,2);
      return time;
    },
    formatTimeCount(remaindTime){
      var hour = parseInt(remaindTime / 3600);
      var min = parseInt((remaindTime % 3600)/60);
      var sec = remaindTime % 60;
      var result = hour + "시간 " + min + "분 " + sec + "초 남았습니다!";
      return result;
    }
  },
  created: function() {
    
    axios.get('/static/products.json').then(response => {
      this.products = response.data.products;
      this.executionCount++;
      for(let i = 0; i < this.products.length; i++) {
        if (this.$route.params.id == this.products[i].id) {
          this.products[i].price = this.$route.params.price;
        }
      }
      console.log(this.products);
      for(let i = 0; i < this.products.length; i++){
      this.reloadRemaindTime(this.products[i]);
      }
    });
  }
}
</script>
<style>
body {
  background-color: #ffffff;
}
#default {
  background-color: #ffffff;
  color : black;
}
</style>

