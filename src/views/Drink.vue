<template>
  <main>
    <button class="btn bg-light mb-2 ml-auto" type="button" 
      @click.prevent="changeState('showEdit', false);changeState('showCreate', true)">
      <span class="fas fa-plus"></span>
      Add New Order
    </button>
    <table class="table">
      <thead>
        <tr>
          <th width="50">刪除</th>
          <th width="50">編輯</th>
          <th width="120" class="click"
            @click="isReverse = !isReverse; changeSortByWay('total')">價格
            <span class="icon isReverse" 
                  :class="{'inverse': isReverse}"
                  v-show="sortBy=='total'"> 
              <i class=" fas fa-angle-up text-secondary h5"></i>
            </span>
          </th>
          <th class="text-left click"
            @click="isReverse = !isReverse; changeSortByWay('name')">名稱
            <span class="icon isReverse" :class="{'inverse': isReverse}"
                  v-show="sortBy=='name'"> 
              <i class=" fas fa-angle-up text-secondary h5"></i>
            </span>
          </th>
          <th class="text-left">內容</th>
          <th width="200">日期
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in sortData" :key="item.key">
          <td class="text-center">
            <button type="button" class="btn btn-sm text-secondary"
              @click.prevent="delOrder(item)">
              <span class="fas fa-trash-alt fa-lg"></span>
            </button>
          </td>
          <td class="text-center">
            <button type="button" class="btn btn-sm"
              @click.prevent="editOrder(item);changeState('showCreate', false);changeState('showEdit', true)">
              <span class="fas fa-pen fa-lg"></span>
            </button>
          </td>
          <td class="text-center">{{ item.total }}</td>
          <td>{{ item.name }}</td>
          <td>
            <ul v-for="(product, i) in item.products" :key="i">
              <li>
                {{ product.title }} * {{ product.qty }} = {{ product.total }}</li>
            </ul>         
          </td>
          <td class="text-center">{{item.date }}</td>
        </tr>
      </tbody>
    </table>
    <div v-if="state.showEdit">
      <h4 class="my-3">編輯訂單</h4>
      <form>
        <div>
          <label for="orderName" class="mr-2">訂單名稱</label>
          <input type="text" id="orderName" name="orderName" v-model="tempOrder.name" placeholder="請輸入你的訂單名稱">
        </div>
        <div v-for="product in tempOrder.products" :key="product.key">
          <label :for="product.name" class="mr-2">{{ product.title }} ${{ product.price }}</label>
          <input :id="product.name" type="number" min="0" max="20" value="1" v-model.number="product.qty">
        </div>
        <button type="submit" class="btn bg-primary text-white"
          @click.prevent="updataOrder()">更新</button>
      </form>
    </div>
    <div v-if="state.showCreate" class="d-flex justify-content-around">
      <div>
        <h4 class="my-3">新增訂單</h4>
        <form>
          <div>
            <label for="orderName" class="mr-2">訂單名稱</label>
            <input type="text" id="orderName" name="orderName" v-model="tempOrder.name" placeholder="請輸入你的訂單名稱">
          </div>
          <div v-for="product in products" :key="product.key">
            <label :for="product.name" class="mr-2">{{ product.title }} ${{ product.price }}</label>
            <input :id="product.name" type="number" min="0" max="20" value="1" v-model.number="product.qty">
            <button type="button" class="btn btn-sm bg-light ml-2"
              @click.prevent="addCart(product.name, product.title, product.price, product.qty)">
              加入
            </button>
          </div>
        </form>
      </div>
      <div class="my-4">
        <h4>我目前的訂單有以下：</h4>
        <ul class="my-4">
          <li v-for="item in Carts" :key="item.key">
            {{ item.title }}： {{ item.qty }} * {{ item.price }}
          </li>
        </ul>
        <button type="submit" class="btn bg-primary text-white"
            :disabled='Carts.length === 0' :class="{ 'disabled': Carts.length === 0 }"
            @click.prevent="addOrder()">新增</button>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      state:{
        showEdit: false,
        showCreate: false,
      },
      orders: [],
      defaultOrders: [
        {
          id: 1,
          name: '我要喝飲料', 
          total: '500',
          date: '2021/5/20',
          products: [
            {
              title: '8 冰綠',
              qty: 2,
              price: 50,
              total: 100,
            },
            {
              title: '鮮柚綠',
              qty: 2,
              price: 50,
              total: 100,
            },
          ]
        },
        {
          id: 2,
          name: '飲料真好喝',
          total: '400',
          date: '2021/5/19'
        },
        {
          id: 3,
          name: '就是要飲料',
          total: '700',
          date: '2021/5/18'
        }
      ],
      products: [
        {
          name: 'milktea',
          title: '要你胖奶茶',
          price: '100',
        },
        {
          name: 'blacktea',
          title: '甜死人紅茶',
          price: '50',
        },
        {
          name: 'greentea',
          title: '年輕人不喝綠茶',
          price: '10',
        },
      ],
      Carts: [],
      tempOrder: {},
      isReverse: false,
      sortBy: 'price',
    }
  },
  methods: {
    changeState(way, boolen) {
      this.state[way] = boolen
    },
    changeSortByWay(way) {
      this.sortBy = way
    },
    delOrder(order) {
      this.orders.splice(this.orders.indexOf(order),1)
      localStorage.setItem('orders', JSON.stringify(this.orders))
    },
    editOrder(item) {
      this.tempOrder = JSON.parse(JSON.stringify(item))
    },
    updataOrder() {
      let index = this.orders.findIndex((item)=> item.id === this.tempOrder.id)
      const date = new Date()
      this.tempOrder.date = `${date.getFullYear()}/${date.getMonth()}/${date.getDate()}`
      this.tempOrder.products = this.tempOrder.products.map((item)=>{
        return {
          ...item,
          total: item.qty * item.price
        }
      })
      this.tempOrder.total = this.tempOrder.products.map(item => item.total).reduce((prev, item) => prev + item);
      this.orders[index] = this.tempOrder
      localStorage.setItem('orders', JSON.stringify(this.orders))
      this.tempOrder = {}
      this.changeState('showEdit', false)
    },
    addOrder() {
      const date = new Date();
      let data = {
        id: Math.max(...this.orders.map(item => item.id)) + 1,
        name: this.tempOrder.name,
        date: `${date.getFullYear()}/${date.getMonth()+1}/${date.getDate()}`,
      };
      data.products = this.Carts.map((item)=>{
        return {
          ...item,
          total: item.qty * item.price
        }
      });
      data.total = data.products.map(item => item.total).reduce((prev, item) => prev + item);
      this.orders.push(data)
      localStorage.setItem('orders', JSON.stringify(this.orders))
      this.tempOrder = {}
      this.Carts = []
      this.changeState('showCreate', false)
    },
    addCart (name, title, price, qty = 0) {
      const data = {
        name, title, price, qty
      }
      this.Carts.push(data)
    },
  },
  // 請在此撰寫 JavaScript
  computed: {
    sortData: function(){
      const vm = this;
      if (!vm.orders) {
        return vm.orders
      } else {
        let newData = vm.orders.sort((a, b)=>{
          let A = a[vm.sortBy]
          let B = b[vm.sortBy]
          if(vm.isReverse){
            return ((A<B)?1:((A>B)?-1:0));
          }else{
            return ((A>B)?1:((A<B)?-1:0));
          }
        })
        return newData
      }
    }
  },
  created () {
    let data = JSON.parse(localStorage.getItem('orders'))
    if(data) {
      this.orders = data;
      } else {
      localStorage.setItem('orders', JSON.stringify(this.defaultOrders))
      this.orders = this.defaultOrders
    }
  }
}
</script>

<style scoped>
th.click {
  cursor: pointer;
}
.icon {
  display: inline-block;
}
.icon.inverse {
  transform: rotate(180deg)
}
</style>
