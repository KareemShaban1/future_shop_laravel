<template>
  <section class="popular-items section">
    <div class="container">
      <div class="row">
        <div class="col-lg-12 mb-2">
          <ul
            class="nav nav-pills profile-pills nav-fill"
            style="direction: rtl"
          >
            <li class="nav-item">
              <a
                v-bind:class="[
                  activeTab == 'profile' ? 'active' : '',
                  'nav-link',
                ]"
                data-toggle="tab"
                href="#profile"
                role="tab"
                @click="changeTab('profile')"
                aria-controls="profile"
                aria-selected="true"
              >
                <i class="icofont-ui-user"></i>
                <span class="d-none d-sm-inline">الملف الشخصى</span></a
              >
            </li>
            <li v-if="user.user_type == 'customer'" class="nav-item">
              <a
                v-bind:class="[
                  activeTab == 'orders' ? 'active' : '',
                  'nav-link',
                ]"
                data-toggle="tab"
                href="#orders"
                role="tab"
                @click="changeTab('orders')"
                aria-controls="orders"
                aria-selected="false"
              >
                <i class="icofont-listine-dots"></i>
                <span class="d-none d-sm-inline">طلباتى</span></a
              >
            </li>
            <li v-if="user.user_type == 'delivery_boy'" class="nav-item">
              <a
                v-bind:class="[
                  activeTab == 'assigned_orders' ? 'active' : '',
                  'nav-link',
                ]"
                data-toggle="tab"
                href="#assigned_orders"
                role="tab"
                @click="changeTab('assigned_orders')"
                aria-controls="assigned_orders"
                aria-selected="false"
                ><i class="icofont-listine-dots"></i>
                <span class="d-none d-sm-inline">{{
                  $lang["My Assigned Orders"]
                }}</span></a
              >
            </li>
            <li v-if="user.user_type == 'delivery_boy'" class="nav-item">
              <a
                v-bind:class="[
                  activeTab == 'delivered_orders' ? 'active' : '',
                  'nav-link',
                ]"
                data-toggle="tab"
                href="#delivered_orders"
                role="tab"
                @click="changeTab('delivered_orders')"
                aria-controls="delivered_orders"
                aria-selected="false"
                ><i class="icofont-listine-dots"></i>
                <span class="d-none d-sm-inline">{{
                  $lang["Delivered Orders"]
                }}</span></a
              >
            </li>
            <li class="nav-item">
              <a
                v-bind:class="[
                  activeTab == 'update_profile' ? 'active' : '',
                  'nav-link',
                ]"
                data-toggle="tab"
                href="#update_profile"
                role="tab"
                @click="changeTab('update_profile')"
                aria-controls="update_profile"
                aria-selected="true"
              >
                <i class="icofont-user-alt-3"></i>
                <span class="d-none d-sm-inline">تعديل الملف الشخصى</span></a
              >
            </li>
            <li class="nav-item">
              <a
                v-bind:class="[
                  activeTab == 'update_profile_picture' ? 'active' : '',
                  'nav-link',
                ]"
                data-toggle="tab"
                href="#update_profile_picture"
                role="tab"
                @click="changeTab('update_profile_picture')"
                aria-controls="update_profile_picture"
                aria-selected="true"
              >
                <i class="icofont-image"></i>
                <span class="d-none d-sm-inline"
                  >تعديل صورة الملف الشخصى</span
                ></a
              >
            </li>
          </ul>
        </div>
        <div class="col-lg-12">
          <div class="tab-content" id="accountTabContent">
            <div
              v-bind:class="[
                activeTab == 'profile' ? 'show active' : '',
                'tab-pane fade',
              ]"
              id="profile"
            >
              <div class="card" style="direction: rtl">
                <div class="card-header" style="text-align: center">
                  <h4 class="header-title">الملف الشخصى</h4>
                </div>
                <div class="card-body" style="text-align: right">
                  <table v-if="user.id" class="table table-bordered">
                    <tbody>
                      <tr>
                        <td>الأسم</td>
                        <td>{{ user.name }}</td>
                      </tr>
                      <tr>
                        <td>البريد الألكترونى</td>
                        <td>{{ user.email }}</td>
                      </tr>
                      <tr>
                        <td>رقم الهاتف</td>
                        <td>{{ user.phone }}</td>
                      </tr>
                      <tr>
                        <td>عنوان التوصيل</td>
                        <td>{{ user.address }}</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>

            <div
              v-if="user.user_type == 'customer'"
              v-bind:class="[
                activeTab == 'orders' ? 'show active' : '',
                'tab-pane fade',
              ]"
              id="orders"
            >
              <MyOrders />
            </div>

            <div
              v-if="
                user.user_type == 'delivery_boy' &&
                activeTab == 'assigned_orders'
              "
              v-bind:class="[
                activeTab == 'assigned_orders' ? 'show active' : '',
                'tab-pane fade',
              ]"
              id="assigned_orders"
            >
              <MyAssignedOrders />
            </div>

            <div
              v-if="
                user.user_type == 'delivery_boy' &&
                activeTab == 'delivered_orders'
              "
              v-bind:class="[
                activeTab == 'delivered_orders' ? 'show active' : '',
                'tab-pane fade',
              ]"
              id="delivered_orders"
            >
              <MyDeliveredOrders />
            </div>

            <div
              v-bind:class="[
                activeTab == 'update_profile' ? 'show active' : '',
                'tab-pane fade',
              ]"
              id="update_profile"
            >
              <div class="card" style="direction: rtl; text-align: right">
                <div class="card-header">
                  <h4 class="header-title text-center">تعديل الملف الشخصى</h4>
                </div>
                <div class="card-body">
                  <form class="form-signup" @submit.prevent="updateMyProfile">
                    <div class="form-label-group mb-3">
                      <label for="name">الأسم *</label>
                      <input
                        type="text"
                        id="name"
                        v-bind:class="
                          errors && errors.name
                            ? 'form-control is-invalid'
                            : 'form-control'
                        "
                        v-model="user.name"
                        placeholder="الأسم"
                      />
                      <span class="text-danger" v-if="errors && errors.name">
                        {{ errors.name[0] }}
                      </span>
                    </div>

                    <div class="form-label-group mb-3">
                      <label for="email">البريد الألكترونى *</label>
                      <input
                        type="email"
                        id="email"
                        v-bind:class="
                          errors && errors.email
                            ? 'form-control is-invalid'
                            : 'form-control'
                        "
                        v-model="user.email"
                        placeholder="البريد الألكترونى"
                      />
                      <span class="text-danger" v-if="errors && errors.email">
                        {{ errors.email[0] }}
                      </span>
                    </div>

                    <div class="form-label-group mb-3">
                      <label for="phone">رقم الهاتف *</label>
                      <input
                        type="text"
                        id="phone"
                        v-bind:class="
                          errors && errors.phone
                            ? 'form-control is-invalid'
                            : 'form-control'
                        "
                        v-model="user.phone"
                        placeholder="رقم الهاتف"
                      />
                      <span class="text-danger" v-if="errors && errors.phone">
                        {{ errors.phone[0] }}
                      </span>
                    </div>

                    <div class="form-label-group mb-3">
                      <label for="phone">عنوان التسليم *</label>
                      <textarea
                        id="address"
                        v-bind:class="
                          errors && errors.address
                            ? 'form-control is-invalid'
                            : 'form-control'
                        "
                        v-model="user.address"
                        placeholder="عنوان التسليم "
                      ></textarea>
                      <span class="text-danger" v-if="errors && errors.address">
                        {{ errors.address[0] }}
                      </span>
                    </div>

                    <div class="form-label-group mb-3">
                      <label for="password">كلمة المرور</label>
                      <input
                        type="password"
                        id="password"
                        v-bind:class="
                          errors && errors.password
                            ? 'form-control is-invalid'
                            : 'form-control'
                        "
                        v-model="user.password"
                        placeholder="كلمة المرور"
                      />
                      <span
                        class="text-danger"
                        v-if="errors && errors.password"
                      >
                        {{ errors.password[0] }}
                      </span>
                    </div>

                    <div class="form-label-group mb-3">
                      <label for="password">تأكيد كلمة المرور</label>
                      <input
                        type="password"
                        id="password_confirmation"
                        class="form-control"
                        v-model="user.password_confirmation"
                        placeholder="تأكيد كلمة المرور"
                      />
                    </div>

                    <div class="d-flex justify-content-center">
                      <button
                        type="submit"
                        class="btn btn-lg text-uppercase"
                        style="background-color: var(--primary-color)"
                      >
                        <span
                          v-if="loading"
                          class="spinner-border spinner-border-sm"
                        ></span>
                        <span v-if="!loading"> حفظ</span>
                      </button>
                    </div>
                  </form>
                </div>
              </div>
            </div>

            <div
              v-bind:class="[
                activeTab == 'update_profile_picture' ? 'show active' : '',
                'tab-pane fade',
              ]"
              id="update_profile_picture"
            >
              <div class="card">
                <div
                  class="card-header d-flex align-items-center justify-content-center"
                >
                  <h4 class="header-title text-center">
                    تعديل صورة الملف الشخصى
                  </h4>
                </div>
                <div class="card-body text-center">
                  <img
                    :src="user.profile_picture"
                    class="img-thumbnail profile-picture"
                  />
                  <span
                    class="text-danger"
                    v-if="errors && errors.profile_picture"
                  >
                    {{ errors.profile_picture[0] }}
                  </span>

                  <input
                    type="file"
                    class="d-none"
                    ref="inputFile"
                    v-on:change="onSelectImage"
                  />
                  <br />

                  <div
                    style="
                      display: flex;
                      align-content: center;
                      flex-direction: column;
                      flex-wrap: wrap;
                      margin-top: 20px;
                    "
                  >
                    <div class="col-md-12">
                      <button
                        type="button"
                        @click="selectFileInput"
                        class="btn text-dark w-75"
                        style="border: 1px solid black"
                      >
                        أختر الصورة
                      </button>
                    </div>

                    <div class="col-md-12">
                      <button
                        type="button"
                        @click="updateMyProfilePicture"
                        class="btn mt-3 w-75"
                        style="background-color: var(--primary-color)"
                      >
                        <span
                          v-if="loading"
                          class="spinner-border spinner-border-sm"
                        ></span>
                        <span v-if="!loading"> حفظ </span>
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import MyOrders from "../components/Orders.vue";
import MyAssignedOrders from "../components/AssignedOrders.vue";
import MyDeliveredOrders from "../components/DeliveredOrders.vue";

export default {
  components: {
    MyOrders,
    MyAssignedOrders,
    MyDeliveredOrders,
  },
  data() {
    return {
      profilePicture: null,
    };
  },
  computed: {
    ...mapGetters("auth", ["user", "loading", "errors"]),
    activeTab() {
      if (this.$route.params.activeTab) {
        return this.$route.params.activeTab;
      }
      return "profile";
    },
  },
  methods: {
    ...mapActions("auth", ["updateProfile", "updateProfilePicture"]),
    changeTab(tab) {
      if (this.$route.params.activeTab != tab) {
        this.$router.replace({ name: "profile", params: { activeTab: tab } });
      }
    },
    selectFileInput(event) {
      this.$refs.inputFile.click();
    },
    onSelectImage(e) {
      this.profilePicture = e.target.files[0];
      this.user.profile_picture = URL.createObjectURL(this.profilePicture);
    },
    updateMyProfile() {
      this.updateProfile().then(() => {
        this.$awn.success(this.$lang["Updated sucessfully"]);
      });
    },
    updateMyProfilePicture() {
      if (this.profilePicture != null) {
        let data = new FormData();
        data.append("profile_picture", this.profilePicture);
        this.updateProfilePicture(data).then(() => {
          this.$awn.success(this.$lang["Updated sucessfully"]);
        });
      }
    },
    logout: function () {
      this.$store.dispatch("auth/logout").then(() => {
        this.$router.push("/login");
      });
    },
  },
};
</script>
