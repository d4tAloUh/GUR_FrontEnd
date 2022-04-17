<template>
  <div>

    <div class="uk-child-width-1-2@m uk-grid">
      <div>
        <div class="uk-card uk-card-default uk-card-small uk-card-body">
          <img src="https://ik.imagekit.io/alouh/misc/cherry-667_h4lXP7VVL.png?ik-sdk-version=javascript-1.4.3&updatedAt=1643909256797" height="500"
               width="500" class="uk-align-center" alt="">
          Illustration by <a href="https://icons8.com/illustrations/author/5e7e24ce01d0360013bb7479">Natasha Remarchuk</a> from <a href="https://icons8.com/illustrations">Ouch!</a>
        </div>
      </div>
      <div>
        <div class="uk-card uk-card-default uk-card-large uk-card-body">

          <form @submit.stop.prevent="handleSubmit">
            <fieldset class="uk-fieldset">

              <legend class="uk-legend">Увійти</legend>
              <div class="uk-align-right">
                <ToggleButton ton label-enable-text="Кур'єр" label-disable-text="Користувач"
                              v-on:change="update_as_courier" v-bind:default-state="as_courier"
                />
              </div>
              <div class="uk-margin">
                <label class="uk-form-label">Емейл</label>
                <input class="uk-input" v-model="username" type="email" placeholder="your.email@gmail.com" required>
              </div>

              <div class="uk-margin">
                <label class="uk-form-label">Пароль</label>
                <input class="uk-input" v-model="password" type="password" required>
              </div>

              <div class="uk-margin">
                <button class="uk-button uk-button-primary uk-width-1-1" :disabled="loading"
                        type="submit">
                  Відправити
                </button>
              </div>

              <div class="uk-margin">
                <p v-if="as_courier">
                  Не маєте аккаунту кур'єра?
                  <NuxtLink :to="{ path: '/courier/register'}" exact>
                    Реєстрація
                  </NuxtLink>
                </p>
                <p v-else>
                  Не маєте аккаунту користувача?
                  <NuxtLink :to="{ path: '/users/register'}" exact>
                    Реєстрація
                  </NuxtLink>
                </p>
              </div>
            </fieldset>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {mapMutations, mapActions, mapGetters} from 'vuex'
import ToggleButton from "~/components/misc/ToggleButton";
import _ from "lodash";

export default {
  name: "signin",
  components: {ToggleButton},
  data: () => ({
    username: '',
    password: '',
    loading: false,
    as_courier: false
  }),
  beforeMount() {
    if (this.$route.params.as_courier){
      this.as_courier = this.$route.params.as_courier
    }
  },
  methods: {
    update_as_courier: function (value) {
      this.as_courier = value
    },
    handleSubmit: _.debounce(async function() {
      try {
        this.loading = true
        await this.login({username:this.username, password:this.password})
        try {
          await this.getUser(this.as_courier)
          this.loading = false
          this.$toast.success("Ви успішно увійшли в свій аккаунт.", {
            toastClassName: ['uk-margin-top']
          })
          if (!this.finishedRegistration) {
            this.$toast.info("Спочатку необхідно закінчити реєстрацію.", {
              toastClassName: ['uk-margin-top']
            })
            if (this.isCourier) {
              await this.$router.push('/courier/register/next')
            } else {
              await this.$router.push('/users/register/next')
            }
          } else {
            if (this.isCourier) {
              await this.$router.push('/courier/')
            } else {
              try{
                await this.syncCart()
              } catch (e){
                this.$toast.info("Сталася помилка, коли отримували замовлення", {
                  toastClassName: ['uk-margin-top']
                })
              }

              await this.$router.push('/profile')
            }
          }
        } catch (err) {
          this.loading = false
          await this.logout()
          this.$toast.error(err.response.data || "Сталася помилка, коли отримували ваш профіль", {
            toastClassName: ['uk-margin-top']
          })
        }
      } catch (err) {
        this.loading = false
        if (!err.response) {
          this.$toast.error("Не отримано відповіді від серверу. Зачекайте, будь ласка.", {
            toastClassName: ['uk-margin-top']
          })
        } else {
          console.error(err.response.data)
          this.$toast.error("Не знайдено такого користувача.", {
            toastClassName: ['uk-margin-top']
          })
        }
      }

    }, 2000, {
      leading: true, trailing: false
    }),
    ...mapActions({
      login: 'authorization/login',
      getUser: 'authorization/getUser',
      syncCart: 'cart/syncWithServer',
      logout: 'authorization/logout'
    })
  },
  computed: {
    ...mapGetters({
      currentUser: 'authorization/getUser',
      finishedRegistration: 'authorization/isSettedUp',
      isCourier: 'authorization/isCourier'
    })
  }
}
</script>

<style scoped>

</style>
