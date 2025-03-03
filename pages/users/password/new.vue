<template>
  <div class="container d-flex cr-vh100">
    <div class="d-flex flex-grow-1 flex-column justify-content-center align-items-center">
      <div :class="[$device.isMobile ? 'h2' : 'h1', 'text-center']">
        {{ $t('login.password.request.title') }}
      </div>
      <p class="form-text text-body text-wrap">
        {{ $t('login.password.request.text') }}
      </p>
      <div class="d-flex flex-column align-items-center w-md-50">
        <form>
          <div class="form-group my-2">
            <input
              id="inputEmail"
              v-model="email"
              type="email"
              class="form-control"
              aria-describedby="emailHelp"
              @input="allowPost"
            >
          </div>
          <div class="d-flex justify-content-end">
            <button
              type="submit"
              class="btn btn-dark my-2 w-100"
              :disabled="disabled"
              @click.stop.prevent="sendRequestEmail"
            >
              {{ $t('login.password.request.submit') }}
            </button>
          </div>
        </form>
        <div class="my-3 d-flex flex-column justify-content-center align-items-center">
          <NuxtLink to="/login">
            {{ $t('login.submit') }}
          </NuxtLink>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'PasswordResetRequest',
  data () {
    return {
      disabled: true,
      email: null,
      errors: []
    }
  },
  methods: {
    allowPost () {
      if (this.email) {
        this.disabled = false
      } else {
        this.disabled = true
      }
    },
    validateEmail (email) {
      const re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      return re.test(String(email).toLowerCase())
    },
    checkForm () {
      this.errors = []
      if (!this.email) {
        this.errors.push(this.$t('signUp.errors.email'))
        return false
      }
      if (!this.validateEmail(this.email)) {
        this.errors.push(this.$t('signUp.errors.emailFormat'))
        return false
      }
      return true
    },
    async sendRequestEmail () {
      const checkForm = this.checkForm()
      if (checkForm) {
        const payload = {
          user: {
            email: this.email
          }
        }
        const response = await this.$store.dispatch('users/authentication/requestPasswordReset', payload)
        if (response.user.token !== null) {
          this.$root.$bvToast.toast(this.$t('login.password.email', { email: response.user.email }), {
            title: 'Cuisinier Rebelle',
            variant: 'info',
            solid: true
          })
          this.email = null
        } else {
          this.$root.$bvToast.toast(this.$t('users.password.errors.exist'), {
            title: 'Cuisinier Rebelle',
            variant: 'danger',
            solid: true
          })
        }
      } else {
        this.$root.$bvToast.toast(this.errors[0], {
          title: 'Cuisinier Rebelle',
          variant: 'danger',
          solid: true
        })
      }
    }
  }
}
</script>
