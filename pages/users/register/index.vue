<template>
  <div>

    <div class="uk-child-width-1-2@m uk-grid">
      <div>
        <div class="uk-card uk-card-default uk-card-small uk-card-body">

          <img src="https://ik.imagekit.io/alouh/misc/bao-delete-confirmation-1_fxUnewRh9W.png?ik-sdk-version=javascript-1.4.3&updatedAt=1643909147188" height="500"
               width="370" class="uk-align-center" alt="">
          Illustration by <a href="https://icons8.com/illustrations/author/60c868b228e02f000aef145d">AsIa Vitalyevna</a> from <a href="https://icons8.com/illustrations">Ouch!</a>
        </div>
      </div>
      <div>
        <div class="uk-card uk-card-default uk-card-large uk-card-body">

          <form @submit.stop.prevent="handleSubmit">
            <fieldset class="uk-fieldset">

              <legend class="uk-legend">Реєстрація користувача</legend>

              <div class="uk-margin">
                <label class="uk-form-label" for="email-id">Емейл</label>
                <input class="uk-input" id="email-id" v-model="email" type="email" placeholder="">
              </div>

              <div class="uk-margin">
                <label class="uk-form-label" for="pass-id">Пароль</label>
                <input class="uk-input" id="pass-id" v-model="password" type="password">
              </div>

              <div class="uk-margin">
                <button class="uk-button uk-button-primary uk-width-1-1" :disabled="loading" type="submit">Надіслати
                </button>
              </div>

              <div class="uk-margin">
                <p>
                  Вже маєте аккаунт?
                  <NuxtLink :to="{ path: '/users/signin'}" exact>
                    Увійти
                  </NuxtLink>
                </p>
              </div>
              <div class="uk-margin">
                <p>
                  Хочете зареєструватися як кур'єр?
                  <NuxtLink :to="{ path: '/courier/register'}" exact>
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
import _ from "lodash";
import ResErrorHandler from "~/utils/ResErrorHandler";

export default {
  name: "register_user",
  data: () => ({
    email: '',
    password: '',
    loading: false,
  }),
  methods: {
    handleSubmit: _.debounce(async function () {
      try {
        this.loading = true;
        this.restaurantList = await this.$axios.$post('/register',
          {
            "email": this.email,
            "password": this.password
          });
        this.loading = false;
        this.$toast.success("Успішна реєстрація, увійдіть у аккаунт.", {
          toastClassName: ['uk-margin-top']
        })
        await this.$router.push('/users/signin')
      } catch (err) {
        this.loading = false;
        if (!err.response)
          this.$toast.warning("Помилка мережі.", {
            toastClassName: ['uk-margin-top']
          })
        else {
          this.$toast.warning(ResErrorHandler.checkFormErrors(err) || "Сталася помилка. Аккаунт не було створено.", {
            toastClassName: ['uk-margin-top']
          })
        }
      }

    }, 2000, {
      leading: true, trailing: false
    })
  },
}
</script>

<style scoped>
</style>
