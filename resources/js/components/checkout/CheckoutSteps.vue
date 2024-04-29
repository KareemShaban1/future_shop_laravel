<template>
  <div>
    <!-- Step 1: Delivery Address -->
    <div>
      <div class="card">
        <div class="card-header">
          <h4 class="text-center">{{ $lang["Delivery Address"] }}</h4>
        </div>
        <div class="card-body">
          <form @submit.prevent="saveAddress" autocomplete="off">
            <div class="form-label-group mb-3">
              <label for="name">{{ $lang["Name"] }} *</label>
              <input
                type="text"
                id="name"
                v-bind:class="
                  errors && errors.name
                    ? 'form-control is-invalid'
                    : 'form-control'
                "
                v-model="user.name"
                :placeholder="$lang['Name']"
                required
              />
              <span class="text-danger" v-if="errors && errors.name">
                {{ errors.name[0] }}
              </span>
            </div>
            <div class="form-label-group mb-3">
              <label for="phone">{{ $lang["Mobile Number"] }} *</label>
              <input
                type="text"
                id="phone"
                v-bind:class="
                  errors && errors.phone
                    ? 'form-control is-invalid'
                    : 'form-control'
                "
                v-model="user.phone"
                :placeholder="$lang['Mobile Number']"
                required
              />
              <span class="text-danger" v-if="errors && errors.phone">
                {{ errors.phone[0] }}
              </span>
            </div>
            <div class="form-label-group mb-3">
              <label for="address">{{ $lang["Shipping Address"] }} *</label>
              <textarea
                id="address"
                v-bind:class="
                  errors && errors.address
                    ? 'form-control is-invalid'
                    : 'form-control'
                "
                v-model="user.address"
                :placeholder="$lang['Shipping Address']"
                required
              ></textarea>
              <span class="text-danger" v-if="errors && errors.address">
                {{ errors.address[0] }}
              </span>
            </div>

            <button type="submit" class="btn btn-primary">
              <span
                v-if="loading"
                class="spinner-border spinner-border-sm"
              ></span>
              <span v-if="!loading">{{ $lang["Next: Delivery Timing"] }}</span>
            </button>
          </form>
        </div>
      </div>
    </div>

    <!-- Step 4: Order Preview -->
    <div>
      <div class="card">
        <div class="card-header">
          <h4 class="text-center">{{ $lang["Order Preview"] }}</h4>
        </div>
        <div class="card-body">
          <form @submit.prevent="makeOrder" autocomplete="off">
            <table class="table">
              <tbody>
                <tr>
                  <td colspan="2" class="text-center">
                    <b>{{ $lang["Delivery Details"] }}</b>
                  </td>
                </tr>
                <tr>
                  <td>{{ $lang["Delivery Type"] }}</td>
                  <td>
                    Express
                  </td>
                </tr>
               
                <tr>
                  <td>{{ $lang["Payment Method"] }}</td>
                  <td>
                    Cash on Delivery
                  </td>
                </tr>
                
              </tbody>
            </table>

            <table class="table">
              <tbody>
                <tr>
                  <td colspan="2" class="text-center">
                    <b>{{ $lang["Order Summary"] }}</b>
                  </td>
                </tr>
                <tr>
                  <td>{{ $lang["Sub Total"] }}</td>
                  <td class="text-right">{{ $currency + " " + subTotal }}</td>
                </tr>
                <tr>
                  <td>{{ $lang["Delivery Charge"] }}</td>
                  <td class="text-right">
                    + {{ $currency + " " + deliveryCharge }}
                  </td>
                </tr>
                <tr>
                  <td>{{ $lang["Discount"] }}</td>
                  <td class="text-right">- {{ $currency + " " + discount }}</td>
                </tr>
                <tr>
                  <td>{{ $lang["Grand Total"] }}</td>
                  <td class="text-right">
                    <b>{{ $currency + " " + grandTotal }}</b>
                  </td>
                </tr>
              </tbody>
            </table>

            <button type="submit" class="btn btn-primary btn-block mt-2">
              <span
                v-if="loading"
                class="spinner-border spinner-border-sm"
              ></span>
              <span v-if="!loading">{{ $lang["Place Order"] }}</span>
            </button>

            <button
              type="button"
              @click="previousStep"
              class="btn btn-light text-dark btn-block"
            >
              {{ $lang["Back"] }}
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import { Datetime } from "vue-datetime";
import moment from "moment";
import "vue-datetime/dist/vue-datetime.css";

export default {
  data() {
    return {
      currentStep: 1,
      order_instructions: "",
      delivery_type: "express",
      delivery_date: new Date().toISOString(),
      minDatetime: new Date().toISOString(),
      payment_method: "cash_on_delivery",
      paypal_active: _paypal_active == "Yes" ? true : false,
      stripe_active: _stripe_active == "Yes" ? true : false,
    };
  },
  components: {
    datetime: Datetime,
  },
  computed: {
    ...mapGetters("auth", ["user", "loading", "errors"]),
    ...mapGetters("checkout", ["errors", "delivery_details"]),
    // deliveryDetails: function () {
    //   return this.$store.getters["checkout/delivery_details"];
    // },
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
    ...mapActions("checkout", [
      "storeAddress",
      "storeTiming",
      "storePaymentMethod",
      "nextStep",
      "previousStep",
      "resetStep",
    ]),
    saveAddress() {
      let data = {
        name: this.user.name,
        phone: this.user.phone,
        address: this.user.address,
      };
      if (this.$store.getters["auth/isLoggedIn"]) {
        this.storeAddress(data).then(() => {
          this.nextStep();
        });
      } else {
        this.$store
          .dispatch("auth/registerGuestUser", data)
          .then(() => {
            this.storeAddress(data).then(() => {
              this.nextStep();
            });
          })
          .catch((error) => {
            console.error("Error checking email:", error);
            this.$awn.alert("An error occurred while checking the email.");
          });
      }
    },
    saveTiming() {
      let data = {
        order_instructions: this.order_instructions,
        delivery_type: this.delivery_type,
        delivery_date: moment(this.delivery_date).format("YYYY-MM-DD HH:mm:ss"),
      };
      this.storeTiming(data).then(() => {
        this.nextStep();
      });
    },
    savePaymentMethod() {
      this.storePaymentMethod(this.payment_method).then(() => {
        this.nextStep();
      });
    },
    makeOrder() {
      let data = {
        cartItems: this.cartItems.flatMap((item) => [
          { id: item.id, qnty: item.qnty },
        ]),
        couponCode: this.couponData ? this.couponData.code : null,
        // deliveryDetails: this.deliveryDetails,
      };
      this.placeOrder(data).then((response) => {
        this.resetStep();
        this.$store.dispatch("cart/emptyCart");
        this.$router.push({
          name: "view_order",
          params: { id: response.data.data.order_id },
        });
        if (this.deliveryDetails.paymentMethod == "cash_on_delivery") {
          this.$awn.success(this.$lang["Your Order placed successfully"]);
        }
      });
    },
  },
  created() {
//     this.order_instructions = this.delivery_details.deliveryTiming
//       ? this.delivery_details.deliveryTiming.order_instructions
//       : "";
//     this.delivery_type = this.delivery_details.deliveryTiming
//       ? this.delivery_details.deliveryTiming.delivery_type
//       : "express";
//     this.delivery_date = this.delivery_details.deliveryTiming
//       ? this.delivery_details.deliveryTiming.delivery_date
//       : new Date().toISOString();
//     this.payment_method = this.delivery_details.paymentMethod
//       ? this.delivery_details.paymentMethod
//       : "cash_on_delivery";
this.delivery_type = "express";
    this.payment_method = "cash_on_delivery";
  },
};
</script>

<style>
/* Add your styles here */
</style>
