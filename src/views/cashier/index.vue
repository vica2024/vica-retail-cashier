<template>
  <div ref="container" class="w-full h-full pb-5">
    <div class="flex h-full pl-5 pr-5">
      <div class="w-1/2 left-box flex h-full overflow-hidden rounded-2xl bg-white">
        <div class="h-full w-full overflow-hidden">
          <div ref="leftContainer">
            <customer-card :customerInfo="customerInfo"/>
            <div class="flex justify-between border-b">
              <span class="ml-5 pb-5">{{ t('selected') }}&nbsp;<b class="text-rose-600">{{ cartState.total_cart }}</b>&nbsp;{{
                  t('items')
                }}</span>
              <a-popconfirm :content="t('clearCartConfirm')" type="warning" @ok="clear_cart" :okText="t('yes')"
                            :cancelText="t('no')">
                <span
                    class="flex items-center cursor-pointer mr-5 pb-5 text-gray-500 hover:text-blue-400 active:text-blue-400">
                  <icon-delete size="16"/> <span class="text-xs">{{ t('clearCart') }}</span>
                </span>
              </a-popconfirm>
            </div>
          </div>
          <div v-if="cartList.length" class="overflow-y-auto" :style="`height:${cartHeight}px;`">
            <div class="pr-5 pl-5 hover:bg-cyan-50" v-for="(item, index) in cartList" :key="index">
              <div class="flex pt-4 pb-4">
                <div class="w-1/5 overflow-hidden">
                  <img alt="" :src="item.cart[0].productInfo.image"/>
                </div>
                <div class="w-4/5 flex-col justify-between pl-2">
                  <div class="h-1/3 flex flex-none justify-between">
                    <div class="pr-2 overflow-hidden text-ellipsis whitespace-nowrap">
                      {{ item.cart[0].productInfo.store_info }}
                    </div>
                    <a-popconfirm :content="t('deleteCartConfirm')" type="warning" @ok="del_goods(item.cart[0])"
                                  :okText="t('yes')" :cancelText="t('no')">
                      <span class="flex whitespace-nowrap cursor-pointer text-blue-300 hover:text-blue-600">
                        <icon-close size="13"/>
                      </span>
                    </a-popconfirm>
                  </div>
                  <div class="h-1/3 w-1/3 cursor-pointer lex-grow flex items-center text-gray-400 text-xs"
                       @click="modalsState.showGoodsAttrsModal=true">
                    {{ item.cart[0].productInfo.attrInfo.suk }}
                    <icon-down/>
                  </div>
                  <div class="h-1/3 flex-none flex place-items-end justify-between">
                    <span class="text-rose-600">
                      <span class="text-xs">{{ t('price') }}</span>
                      <b class="text-base">{{ item.cart[0].productInfo.price }}</b>
                    </span>
                    <div class="flex items-center justify-center cart-item">
                      <a-input-number v-model="item.cart[0].cart_num" @change="fetchChangeCart(item.cart[0])"
                                      size="mini" mode="button"
                                      :min="1"
                                      class="bg-white text-center" style="width:80px;border-radius:14px;"/>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="w-full h-3/4 flex items-center" :style="`height:${cartHeight}px;`" v-else>
            <div class="w-3/5 m-auto text-center">
              <img alt="" src="../../assets/images/no-cart.png">
              <span class="text-gray-400 text-xs">{{ t('noItems') }}</span>
            </div>
          </div>
          <div class="w-full shaw-1" :style="`height:${cartPaymentHeight}px;`">
            <div class="flex justify-between border-b">
              <div class="cursor-pointer font-bold p-3 flex items-center active:text-blue-400"
                   @click="modalsState.showSalesPerson=true">
                <span class="pr-1 lg:text-base sm:text-md">{{ cashierState.currentStaff.staff_name }}</span>
                <icon-down/>
              </div>
              <div class="flex justify-between items-center text-sm pr-3">
                  <span class="flex items-center text-sm">{{ t('total') }}:
                        <span class="text-rose-600">{{ t('price') }}</span>
                       <b class="text-rose-600 font-bold text-lg">{{ cartState.payPrice }}</b>&nbsp;
                       <a-popconfirm content="Do you want to discard the draft?" okText="Close" cancelText=" ">
                          <icon-question-circle class="cursor-pointer hover:text-blue-600" size="14"/>
                          <template #content>
                              <h3 class="text-center font-bold mb-3 border-b">{{ t('details') }}</h3>
                              <p class="mb-1">{{ t('originalPrice', {price: cartState.payPrice}) }}</p>
                              <p class="mb-1">{{ t('memberDiscount', {discount: 0}) }}</p>
                              <p class="mb-1">{{ t('couponAmount', {amount: 0}) }}</p>
                              <p class="mb-1">{{ t('pointsDeduction', {points: 0}) }}</p>
                              <p>{{ t('limitedTimeDiscount', {discount: 0}) }}</p>
                        </template>
                      </a-popconfirm>
                </span>
              </div>
            </div>
            <div class="p-5">
              <a-button long type="primary" class="h-11" shape="round" @click="modalsState.showPayment=true">
                <span class="text-base font-bold">{{ t('checkout') }}</span>
              </a-button>
            </div>
          </div>
        </div>
        <div class="pt-5 pl-3 pr-3 border-l">
          <counter-tools :stages="toolsState"/>
        </div>
      </div>
      <div class="w-1/2 h-full ml-5">
        <div class="w-full bg-white p-5 mb-5 rounded-2xl">
          <div class="flex overflow-hidden w-full">
            <a-auto-complete v-model="searchText"
                             :data="highlightedOptions"
                             @search="handleSearch"
                             :allow-clear="true"
                             :strict="true"
                             :placeholder="t('searchPlaceholder')"
                             class="bg-white rounded-tl-3xl rounded-bl-3xl border-tlb-primary pt-1 pb-1">
              <template #option="{ data }">
                <div class="flex items-center text-xs pt-3 pb-3">
                  <span class="text-gray-500">手机号:&nbsp; </span>
                  <highlight-words :search-words="[searchText]" :text-to-highlight="data.raw.label"
                                   highlight-class="text-sm"/>
                  <span class="ml-2 text-gray-500">姓名:wayen zhuo</span>
                </div>
              </template>
            </a-auto-complete>
            <a-button type="primary" style="height: 41px;" class="pl-5 pr-5 rounded-tr-3xl rounded-br-3xl">
              <icon-search size="18"/>
            </a-button>
          </div>
          <div class="w-full mt-3 flex">
            <a-select :bordered="false" :placeholder="t('categoryPlaceholder')">
              <a-option v-for="(option, index) in cateList" :key="index">{{ option.cate_name }}</a-option>
            </a-select>
            <a-select :bordered="false" :placeholder="t('hotSales')">
              <a-option>{{ t('weeklyHot') }}</a-option>
              <a-option>{{ t('monthlyHot') }}</a-option>
              <a-option>{{ t('yearlyHot') }}</a-option>
            </a-select>
          </div>
        </div>
        <div class="overflow-y-auto overflow-hidden rounded-2xl" style="height: 68.7vh;">
          <a-row :gutter="12">
            <a-col class="goodsCard" v-for="(item, index) in goodsList" :key="index" :xs="{ span: 24, offset: 0 }"
                   :lg="{ span: 8, offset: 0 }" :md="{ span: 12, offset: 0 }">
              <a-card class="rounded-2xl overflow-hidden p-3 mb-4 hover:bg-blue-600" :body-style="{'padding':0}"
                      :bordered="false">
                <template #cover>
                  <div class="rounded-lg overflow-hidden">
                    <a-image style="width: 100%;" :src="item.image"/>
                    <div v-if="item.stock === 0" class="absolute top-0 left-0 bg-gray-600 text-blue-50 trip text-xs">
                      {{ t('soldOut') }}
                    </div>
                  </div>
                </template>
                <div class="font-500 cursor-pointer pr-1"
                     @click="openGoodsAttr(item)">
                  <div class="overflow-hidden text-ellipsis whitespace-nowrap goodsName">{{ item.store_name }}</div>
                  <div class="text-lg font-bold text-rose-600 goodsPrice">{{ t('price') }}{{ item.price }}</div>
                </div>
              </a-card>
            </a-col>
          </a-row>
        </div>
      </div>
    </div>
    <drawer-payment  :modalTitle="t('paymentType')"/>
    <modal-goods-attr v-if="productAttrs.goodsName" :modalTitle="t('goodsAttr')" :productAttrs="productAttrs"/>
    <!--    <modal-customer-list v-if="modalsState.showCustomerModal" :customerList="customerList"/>-->
    <modal-salesperson :modalTitle="t('modalSalespersonTitle')"
                       :staff-list="cashierState.staffList"/>
    <modal-sales-mark :modalTitle="t('remark')"/>
    <modal-modify-price  :modalTitle="t('remark')"/>
    <modal-recharge :modalTitle="t('remark')" :recharges="rechargeList"/>
    <drawer-customer-search/>
    <!--  customer create drawer  -->
    <drawer-customer-create />
  </div>
</template>
<script setup>
import {ref, provide, watch, reactive, onMounted, nextTick} from "vue";
import {cashierProduct, cashierCate, cashierDetail} from '@/api/product.js';
import ModalGoodsAttr from "./components/modal-goods-attr.vue";
import {cashierCart, cashierCartDel, cashierCartList, cashierCartNum, cashierCompute} from "@/api/order.js";
import CustomerCard from "@/views/cashier/components/customer-card.vue";
import DrawerPayment from "@/views/cashier/components/drawer-payment.vue";
import {infoApi, cashierList} from "@/api/user.js";
import ModalSalesMark from "@/views/cashier/components/modal-sales-mark.vue";
import ModalSalesperson from "@/views/cashier/components/modal-salesperson.vue";
import CounterTools from "@/views/cashier/components/counter-tools.vue";
import ModalModifyPrice from "@/views/cashier/components/modal-modify-price.vue";
import ModalRecharge from "@/views/cashier/components/modal-recharge.vue";
import {getRechargeInfo} from "@/api/store.js";
import {Message} from '@arco-design/web-vue';
import {useI18n} from "vue-i18n";
import HighlightWords from "vue-highlight-words";
import {customerListSearch} from "@/api/customer.js";
import DrawerCustomerSearch from "@/views/customer/components/drawer-customer-search.vue";
import DrawerCustomerCreate from "@/views/customer/components/drawer-customer-create.vue";

const {t} = useI18n();

const container = ref(null);

const leftContainer = ref(null);

//current cart ids
const cartIds = ref([]);

// Query conditions
const searchCondition = reactive({
  store_name: null,
  field_key: 'all',
  cate_id: null,
  page: 1,
  limit: 30,
  uid: 22690,
  staff_id: 4
});
// Product list
const goodsList = ref([]);
// Product categories
const cateList = ref([]);
// Shopping cart list
const cartList = ref([]);
// Customer list
const customerList = ref([]);
//recharge info form backend.
const rechargeList = ref([]);

//using time as a new one.
const tourist_uid = ref(Date.now());

// Current customer
const customerInfo = ref({
  uid: 0,
  nickname: '--',
  now_money: 0,
  integral: 0
});

// Cashier staff data
const cashierState = reactive({
  count: 0,
  currentStaff: {},
  staffList: [],
});

// Product attributes
const productAttrs = reactive({
  attrs: {},
  stock: 0,
  price: 0,
  imageUrl: null,
  goodsName: null,
  isSkill: false,
});

//tools status
const toolsState = reactive({
  pendingOrder: true,
  recharge: true,
  points: true,
  coupon: true,
  changePrice: true,
  mark: true,
})

// Modal states
const modalsState = reactive({
  showGoodsAttrsModal: false,
  showPayment: false,
  showSalesPerson: false,
  showPriceDetail: false,
  showRecharge: false,
  showModifyPrice: false,
  showMark: false,
  showCustomerSearchModal: false,
  showCustomerCreate: false,
});

const toolsWidth = ref(0);
const cartWidth = ref(0);
// Cart height
const cartHeight = ref(0);
// Payment button height
const cartPaymentHeight = ref(170);

const cartState = reactive({
  total_cart: 0,
  payPrice: 0
});

onMounted(async () => {
  await nextTick();
  if (container.value) {
    cartHeight.value = container.value.clientHeight - (leftContainer.value.clientHeight + cartPaymentHeight.value)
    cartWidth.value = Math.round((leftContainer.value.clientWidth / 100) * 67);
    toolsWidth.value = Math.round((leftContainer.value.clientWidth / 100) * 33);
  }
});

watch(
    () => modalsState.showCustomerSearchModal,
    (newVal, oldVal) => {
      if (newVal) {
      }
    }
);

const setModalVisibility = (modalName, visible) => {
  modalsState[modalName] = visible;
};

//To empty the cart.
const clear_cart = () => {
  fetchData(() => cashierCartDel(customerInfo.value.uid, {ids: cartIds.value}), (data, msg) => {
    cartIds.value = [];
    // get cart newly list again.
    fetchCashierCartList();
    Message.success(msg);
  });
}

// To delete one item in cart
const del_goods = (item) => {
  let ids = cartIds.value;
  console.log(ids, item.id)
  fetchData(() => cashierCartDel(customerInfo.value.uid, {ids: [item.id]}), (data, msg) => {
    let index = ids.indexOf(item.id);
    if (index !== -1) {
      cartIds.value.splice(index, 1);
    }
    //console.log(cartIds.value)
    // get cart newly list again.
    fetchCashierCartList();
    Message.success(msg);
  });
}

//To create a new cart and add a goods to the cart.
const addCart = (attrs) => {
  attrs.staff_id = cashierState.currentStaff.id;
  if (customerInfo.value.uid === 0) {
    attrs.tourist_uid = tourist_uid;
  }
  fetchData(() => cashierCart(customerInfo.value.uid, attrs), (data, msg) => {
    if (data.cartId) {
      Message.success(msg)
      !cartIds.value.includes(data.cartId) ? cartIds.value.push(data.cartId) : null;
      //console.log(cartIds.value);
      // get cart newly list again.
      fetchCashierCartList();
    }

  });
}

//get a attrs of the product and open a modal
const openGoodsAttr = (product) => {
  //console.log(product)
  fetchData(() => cashierDetail(product.id, customerInfo.value.uid), data => {
    productAttrs.goodsName = data.storeInfo.store_name;
    productAttrs.imageUrl = data.storeInfo.image;
    productAttrs.price = data.storeInfo.price;
    productAttrs.stock = data.storeInfo.stock;
    productAttrs.attrs = data.productAttr;
    productAttrs.productValue = data.productValue;
    modalsState.showGoodsAttrsModal = true
    //console.log(data);
  });
}

const setNewCustomer = async (map) => {
  const {data} = await customerListSearch(map);
  console.log(data);
}

const getCustomerList = async (map) => {
  const {data} = await customerListSearch(map);
  return data;
}


//Set up a new clerk to make a sale
const setNewStaff = (staff) => {
  modalsState.showSalesPerson = false;
  cashierState.currentStaff = staff;
  //to get cart list again base new staff or new customer
  fetchCashierCartList();

}

// Generic function to handle API calls
const fetchData = async (fetchFunction, successCallback, errorCallback = null) => {
  try {
    const {data, msg} = await fetchFunction();
    successCallback(data, msg);
  } catch (err) {
    if (errorCallback) {
      errorCallback(err);
    } else {
      // Default error handling
      console.error(err);
    }
  } finally {
    // Final block to be executed
  }
};

// Fetch product list and update the goods list
const fetchCashierProduct = () => {
  fetchData(() => cashierProduct(searchCondition), data => {
    goodsList.value = data.list;
    //console.log(data.list);
  });
}

//get recharge data for api
const fetchRechargeList = () => {
  fetchData(getRechargeInfo, data => {
    rechargeList.value = data.recharge_quota;
    //console.log(data)
  });

}

//to change the item quality in the cart.
const fetchChangeCart = (item) => {
  fetchData(() => cashierCartNum(customerInfo.value.uid, {id: item.id, number: item.cart_num}), data => {
    fetchCashierCartList();
  });
};

// Fetch product categories and update the category list
const fetchProductCategory = () => {
  fetchData(cashierCate, data => {
    cateList.value = data;
  });
};

// Fetch cart list and update the cart state
const fetchCashierCartList = () => {
  const customer_id = customerInfo.value.uid;
  const staff_id = cashierState.currentStaff.id;
  let maps = {}
  if (customer_id === 0) {
    maps.tourist_uid = tourist_uid.value;
  }
  fetchData(() => cashierCartList(customer_id, staff_id, maps), async res => {

    cartList.value = res.valid && res.valid.length > 0 ? res.valid : [];
    cartState.total_cart = res.count;

    if (res.count > 0) {
      //enable mark of order
      toolsState.mark = false;
      //enable pending order
      toolsState.pendingOrder = false;

      fetchCashierCompute();
    } else {
      cartState.payPrice = 0;
    }
  });
};

//To calculate the cart price.
const fetchCashierCompute = () => {
  fetchData(() => cashierCompute(customerInfo.value.uid, {
    cart_id: cartIds.value,
    coupon: false,
    coupon_id: 0,
    integral: false
  }), data => {
    cartState.payPrice = data.payPrice;
    //console.log(data)
  });
}


// Fetch customer list and update the customer list state
const fetchCustomerList = () => {

};

// Fetch cashier list and update the cashier state
const fetchCashierList = () => {
  const map = {
    keyword: null,
    page: 1,
    limit: 12
  }
  fetchData(() => cashierList(map), data => {
    cashierState.count = data.count;
    cashierState.currentStaff = data.staffInfo;
    cashierState.staffList = data.staffList;
    //get cart list base on current staff id;
    fetchCashierCartList();
  });
};

// To get a record with current cutomer
const fetchCustomerCartRecord = () => {
  if (customerInfo.value.uid === 0) {
    return false;
  }
  const params = {}
  fetchData(() => infoApi(params), data => {
    console.log(data)
  });
};

// Fetch cashier list and update the cashier state
const fetchSwitchCustomer = () => {
  fetchData(infoApi, data => {

  });
};

// Use Promise.all to call all fetch functions in parallel
const fetchAllData = async () => {
  await Promise.all([
    fetchProductCategory(),
    fetchCustomerCartRecord(),
    fetchCashierProduct(),
    fetchCashierList(),
    fetchRechargeList()
  ]);
};

provide('modalsState', modalsState);
provide('setModalVisibility', setModalVisibility);

provide('setNewStaff', setNewStaff);
provide('setNewCustomer', setNewCustomer);
provide('getCustomerList', getCustomerList);
provide('addCart', addCart);

// Call the function to fetch all data
fetchAllData();


</script>

<style scoped lang="less">
.left-box {
  min-width: 380px;
}

.goodsCard {
  :hover {
    color: #FFFFFF;

    .goodsName {
      color: #FFFFFF
    }

    .goodsPrice {
      color: #FFFFFF
    }
  }
}

.trip {
  display: flex;
  padding: 5px 10px;
  align-items: center;
  border-radius: 0 0 10px 0;
  flex-direction: column;
  justify-content: center;
}

</style>
