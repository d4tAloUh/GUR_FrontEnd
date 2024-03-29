<template>
  <div class="uk-card uk-card-default uk-margin">
    <div v-if="$fetchState.pending">
      <Loading/>
    </div>
    <div v-else class="uk-card-body">
      <h3>Замовлення № {{ order_id }}</h3>

      <div>Сума: {{ decimalPrice(order.summary) }}₴</div>
      <div>Адреса ресторану: {{ order.restaurant.rest_address }}</div>
      <div>Адреса доставки: {{ order.delivery_address }}</div>
      <div v-if="profile">
        <div>Ім'я замовника: {{ profile.first_name }}</div>
        <div>Номер для зв'язку: <a v-bind:href="`tel:+` + profile.tel_num">{{ profile.tel_num }}</a></div>
      </div>
      <div>
        <button v-if="!showDetails" v-on:click="toggleDetails" class="uk-button uk-margin-top uk-margin-bottom">
          Показати деталі
        </button>
        <button v-else v-on:click="toggleDetails" class="uk-button uk-margin-top uk-margin-bottom">Приховати деталі
        </button>
      </div>
      <div v-if="showDetails">
        <div>Ресторан: {{ order.restaurant.name }}</div>
        <div>Відстань до ресторану: ~{{ haversine_distance(order.restaurant.location, {longitude, latitude}) }} км
        </div>
        <div>Відстань від замовлення до ресторану:
          ~{{ haversine_distance(order.delivery_location, order.restaurant.location) }} км
        </div>
        <div class="uk-margin-top">
          <table class="uk-table uk-table-divider">
            <caption><h5>Страви</h5></caption>
            <thead>
            <tr>
              <th>Назва</th>
              <th>Ціна</th>
              <th>Кількість</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="dish in dishes">
              <td class="uk-width-1-2">{{ dish.name }}</td>
              <td class="uk-table-shrink">{{ decimalPrice(dish.price) }}₴</td>
              <td class="uk-table-shrink">{{ dish.quantity }}</td>
            </tr>
            </tbody>
          </table>
        </div>
        <div v-if="order.order_details.length !== 0">Примітки: {{ order.order_details }}</div>
      </div>
      <button class="uk-button green uk-margin-top" @click="finish_order">Доставлено</button>
      <button class="uk-button uk-button-danger uk-margin-top" @click="cancel_order">Відмінити замовлення</button>
    </div>
  </div>
</template>

<script>
import auth from "~/middleware/auth";
import setted from "~/middleware/setted";
import {mapActions, mapGetters} from "vuex";
import OrderHelper from "@/utils/OrderHelper";
import Loading from "@/components/misc/LoadingBar";

export default {
  name: "CurrentOrder",
  middleware: [auth, setted],
  components: {
    Loading
  },
  data: () => ({
    interval: null,
    showDetails: false,
  }),
  async fetch() {
    if (!this.dishes){
      await this.getDetails()
    }
  },
  mounted() {
    this.interval = setInterval(this.send_update, 5000)
  },
  deactivated() {
    clearInterval(this.interval)
  },
  beforeDestroy() {
    clearInterval(this.interval)
  },
  methods: {
    toggleDetails() {
      this.showDetails = !this.showDetails
    },
    decimalPrice: OrderHelper.decimalPrice,
    haversine_distance: OrderHelper.haversine_distance,
    async getDetails() {
      try {
        let response = await this.$axios.$get('/courier-orders/' + this.order_id);
        this.dishes = response.dishes
        this.order = response.order
        this.profile = response.profile
      } catch (err) {
        if (!err.response) {
          this.$toast.error("Помилка мережі", {
            toastClassName: ['uk-margin-top']
          })
          console.error(err)
        } else {
          if (Number(err.response.status) === 404) {
            this.$toast.error("Такого замовлення не існує", {
              toastClassName: ['uk-margin-top']
            })
            await this.$router.push('/profile')
          } else {
            await this.$router.push('/profile')

            this.$toast.error("Сталася помилка", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err.response.data)
          }
        }
      }
    },
    async send_update() {
      if (this.order_exists) {
        try {
          await this.$axios.$post('/courier/location/' + this.order_id, {
            location: this.location
          });
        } catch (err) {
          if (!err.response) {
            this.$toast.error("Помилка мережі", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err)
          } else {
            this.$toast.error(err.response.data.location && err.response.data.location[0] || err.response.data.error || "Сталася помилка", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err.response)
          }
        }
      } else {
        this.$toast.info("Ви нічого наразі не доставляєте", {
          toastClassName: ['uk-margin-top']
        })
      }
    },
    async finish_order() {
      if (this.order_exists) {
        try {
          await this.$axios.$post('/order-statuses/' + this.order_id, {
            status: "F",
          });
          await this.clear_order("Дякуємо за доставлене замовлення")
        } catch (err) {
          if (!err.response) {
            this.$toast.error("Помилка мережі", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err)
          } else {
            this.$toast.error(err.response.data.error || "Сталася помилка", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err.response)
          }
        }
      } else {
        this.$toast.info("Ви нічого наразі не доставляєте", {
          toastClassName: ['uk-margin-top']
        })
      }
    },
    async cancel_order() {
      if (this.order_exists) {
        try {
          await this.$axios.$post('/order-statuses/' + this.order_id, {
            status: "C"
          });
          await this.clear_order("Замовлення було скасовано")
        } catch (err) {
          if (!err.response) {
            this.$toast.error("Помилка мережі", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err)
          } else {
            this.$toast.error(err.response.data.error || "Сталася помилка", {
              toastClassName: ['uk-margin-top']
            })
            console.error(err.response)
          }
        }
      } else {
        this.$toast.info("Ви нічого наразі не доставляєте", {
          toastClassName: ['uk-margin-top']
        })
      }
    },
    async clear_order(content) {
      await (this.order = null)
      await (this.dishes = null)
      await (this.profile = null)
      clearInterval(this.interval)
      this.$toast.info(content, {
        toastClassName: ['uk-margin-top']
      })
    }
  },
  computed: {
    ...mapGetters({
      order_id: 'courier/order_id',
      order_exists: 'courier/order_exists',
      courier_working: 'courier/courier_working',
      location: 'courier/courier_location',
    }),
    order: {
      get() {
        return this.$store.getters['courier/order']
      },
      set(value) {
        this.$store.dispatch('courier/do_set_order', value)
      }
    },
    dishes: {
      get() {
        return this.$store.getters['courier/order_dishes']
      },
      set(value) {
        this.$store.dispatch('courier/do_set_order_dishes', value)
      }
    },
    profile: {
      get() {
        return this.$store.getters['courier/order_profile']
      },
      set(value) {
        this.$store.dispatch('courier/do_set_order_profile', value)
      }
    },
    longitude: {
      get() {
        return this.$store.getters['courier/courier_location'].longitude
      },
      set(value) {
        this.$store.dispatch('courier/do_set_courier_longitude', value)
      }
    },
    latitude: {
      get() {
        return this.$store.getters['courier/courier_location'].latitude
      },
      set(value) {
        this.$store.dispatch('courier/do_set_courier_latitude', value)
      }
    },
  },
}
</script>

<style scoped>
.green {
  background: green;
  color: white;
}
</style>
