<template>
  <div class="card">
    <div class="card-header">
      <h4 class="text-center">
        معاينة الطلب
      </h4>
    </div>
    <div class="card-body" style="direction:rtl">
      <form @submit.prevent="makeOrder" autocomplete="off">
        <table class="table" style="text-align: right; ">
          <tbody>
            <tr>
              <td colspan="2" class="text-center">
                <b>
                  تفاصيل التسليم
                </b>
              </td>
            </tr>
            <tr>
              <td>
                عنوان التسليم
              </td>
              <td>{{ deliveryDetails ? deliveryDetails.deliveryAddress.address : '' }}</td>
            </tr>
            <tr>
              <td>
                نوع التوصيل
              </td>
              <td>
                Express
              </td>
            </tr>
            
            <tr>
              <td>
                طريقة الدفع او السداد
              </td>
              <td>
                Cash On Delivery
              </td>
            </tr>
        
          </tbody>
        </table>

        <table class="table" style="text-align: right; ">
          <tbody>
            <tr>
              <td colspan="2" class="text-center">
                <b>
                  ملخص الطلب
                </b>
              </td>
            </tr>
            <tr>
              <td>
                المجموع الفرعي
              </td>
              <td class="text-right">{{ $currency + " " + subTotal }}</td>
            </tr>
            <tr>
              <td>
                <!-- {{ $lang["Delivery Charge"] }} -->
                رسوم التوصيل
              </td>
              <td class="text-right">
                + {{ $currency + " " + deliveryCharge }}
              </td>
            </tr>
            <tr>
              <td>
                <!-- {{ $lang["Discount"] }} -->
                الخصم
              </td>
              <td class="text-right">- {{ $currency + " " + discount }}</td>
            </tr>
            <tr>
              <td>
                <!-- {{ $lang["Grand Total"] }} -->
                المجموع الإجمالي
              </td>
              <td class="text-right">
                <b>{{ $currency + " " + grandTotal }}</b>
              </td>
            </tr>
          </tbody>
        </table>

        <button type="submit" class="btn btn-block mt-2" style="background-color:var(--primary-color)">
          <span v-if="loading" class="spinner-border spinner-border-sm"></span>
          <span v-if="!loading">
            اطلب الأن
          </span>
        </button>

        <button
          type="button"
          @click="previousStep"
          class="btn btn-light text-dark btn-block"
        >
          رجوع
        </button>
      </form>
    </div>
  </div>
</template>

<script>
import { mapActions } from "vuex";
import moment from "moment";

export default {
  computed: {
    loading: function () {
      return this.$store.getters["orders/loading"];
    },
    deliveryDetails: function () {
      return this.$store.getters["checkout/delivery_details"];
    },
    // deliveryDate: function () {
    //   return moment(this.deliveryDetails.deliveryTiming.delivery_date).format(
    //     "MMM DD, YYYY hh:mm a"
    //   );
    // },
    cartItems: function () {
      return this.$store.getters["cart/cartItems"];
    },
    couponData: function () {
      return this.$store.getters["cart/couponData"];
    },
    subTotal: function () {
      return this.cartItems
        .reduce(
          (s_total, item) => s_total + item.qnty * item.product.current_price,
          0
        )
        .toFixed(2);
    },
    deliveryCharge: function () {
      const deliveryCharge = this.subTotal > 0 ? this.$deliveryCharge : 0;
      if (
        this.$freeDeliveryAmount > 0 &&
        this.subTotal > parseFloat(this.$freeDeliveryAmount)
      ) {
        return parseFloat(0).toFixed(2);
      }
      return parseFloat(deliveryCharge).toFixed(2);
    },
    discount: function () {
      return parseFloat(this.$store.getters["cart/discount"]).toFixed(2);
    },
    grandTotal: function () {
      const total =
        parseFloat(this.subTotal) +
        parseFloat(this.deliveryCharge) -
        parseFloat(this.discount);
      return total.toFixed(2);
    },
  },
  methods: {
    ...mapActions("checkout", ["resetStep", "previousStep"]),
    ...mapActions("orders", ["placeOrder"]),
    makeOrder() {
      let data = {
        cartItems: this.cartItems.flatMap((item) => [
          { id: item.id, qnty: item.qnty },
        ]),
        couponCode: this.couponData ? this.couponData.code : null,
        deliveryDetails: {
      ...this.deliveryDetails,
      deliveryTiming: {
        order_instructions: "", 
        delivery_type: "express",
        delivery_date: moment(new Date().toISOString()).format("MMM DD, YYYY hh:mm a"),
      },
      paymentMethod: "cash_on_delivery",
    },
      };


      this.placeOrder(data).then((response) => {
        this.resetStep();
        this.$store.dispatch("cart/emptyCart");
        this.$router.push({
          name: "view_order",
          params: { id: response.data.data.order_id },
        });
        // if (this.deliveryDetails.paymentMethod == "cash_on_delivery") {
        //   this.$awn.success(this.$lang["Your Order placed successfully"]);
        // }
      });
    },
  },
};
</script>
