<template>
  <div class="uk-card uk-card-default uk-card-body uk-margin" uk-sticky="offset: 20; bottom: true">
    <img src="https://ik.imagekit.io/alouh/misc/twinkle-woman-standing-with-a-shopping-cart_TdYTFGd0L.png?ik-sdk-version=javascript-1.4.3&updatedAt=1650221625449"
         class="uk-align-center" height="250" width="250" alt=""/>

    <div v-if="price > 0">

      <table class="uk-table uk-table-striped uk-table-small uk-table-responsive">
        <thead>
        <tr>
          <th>Назва</th>
          <th>Ціна (шт.)</th>
          <th>Кількість</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="dish in selectedDishes">
          <td class="uk-width-1-2">{{ dish.name }}</td>
          <td class="uk-table-shrink">{{ decimalPrice(dish.price) }}₴</td>
          <td class="uk-table-shrink">{{ dish.quantity }}</td>
          <td>
            <a class="uk-margin-left"><span class="uk-badge" @click="addToCart(dish)">+</span></a>
            <a><span class="uk-badge" style="background: #ff9d03;" @click="removeFromCart(dish)">-</span></a>
            <a><span class="uk-badge" style="background: #f0506e;" @click="deleteFromCart(dish)">Видалити</span></a>
          </td>

        </tr>
        </tbody>
      </table>

      <NuxtLink tag="button" to="/users/orders/create" class="uk-button uk-button-primary" name="button">Зробити замовлення
        ({{ decimalPrice(price) }}₴)
      </NuxtLink>
    </div>
    <div v-else>
      Ви поки нічого не додали в кошик.
    </div>

  </div>
</template>

<script>
import {mapActions} from 'vuex'
import OrderHelper from "~/utils/OrderHelper"

export default {
  name: "Cart",
  methods: {
    async addToCart(item) {
      this.$store.dispatch('cart/addItem', item)
        .catch(err => console.error(err))
    },
    ...mapActions({
      removeFromCart: 'cart/removeItem',
      deleteFromCart: 'cart/deleteItem',
      syncOrder: 'cart/syncWithServer'
    }),
    decimalPrice: OrderHelper.decimalPrice
  },
  async beforeMount() {
    await this.syncOrder();
  },
  computed: {
    id() {
      return this.$route.params.id
    },
    selectedDishes() {
      return this.$store.getters['cart/items']
    },
    price() {
      return this.$store.getters['cart/price']
    },

  }
}
</script>

<style scoped>

</style>
