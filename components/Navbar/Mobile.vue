<template>
  <div
    :key="componentKey"
    ref="navbar"
    v-click-outside="collapse"
    :class="['navbar fixed-top d-flex flex-column px-3 py-2 bg-white']"
  >
    <div class="d-flex w-100 justify-content-between align-items-center">
      <div class="d-flex align-items-center">
        <NuxtLink
          to="/"
          class="navbar-brand d-flex align-items-center text-body"
          @click.native="scroll2Top"
        >
          <img :src="'https://media.cuisinierrebelle.com/brand-icon.jpg'" width="32" height="32" class="mr-1">
          <span>{{ $t('navbar.brand') }}</span>
        </NuxtLink>
      </div>
      <div class="d-flex align-items-center">
        <NuxtLink
          v-if="!show"
          to="/notifications"
          class="nav-item mx-2 text-body text-decoration-none"
          @click.native="getNotifications"
        >
          <i class="material-icons md-24 d-flex">{{ icons.notifications }}</i>
        </NuxtLink>
        <div @click="collapseMenu">
          <i class="material-icons md-24 d-flex">{{ iconMenu }}</i>
        </div>
      </div>
    </div>
    <transition name="fade">
      <div v-if="show" class="mt-2 d-flex flex-column w-100 vh-100">
        <div v-if="isAuthenticated" class="d-flex flex-column">
          <NuxtLink
            :to="`/u/${ currentUser.slug }`"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.recipes') }}
          </NuxtLink>
          <NuxtLink
            v-if="currentUser.bookmarks.length > 0"
            to="/bookmarks"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.bookmarks') }}
          </NuxtLink>
          <NuxtLink
            v-if="currentUser.following.data.length > 0"
            :to="`/u/${ currentUser.slug }/following`"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.following') }}
          </NuxtLink>
          <NuxtLink
            to="/r/new"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.new_recipe') }}
          </NuxtLink>
          <NuxtLink
            to="/top100"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            Top 100
          </NuxtLink>
          <NuxtLink
            :to="`/u/${ currentUser.slug }/settings`"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.settings') }}
          </NuxtLink>
          <NuxtLink
            to="/products"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.products') }}
          </NuxtLink>
          <div
            class="text-body lead my-2 text-decoration-none"
            @click="logout"
          >
            {{ $t('navbar.logout') }}
          </div>
        </div>
        <div v-else class="d-flex flex-column">
          <NuxtLink
            to="/products"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.products') }}
          </NuxtLink>
          <NuxtLink
            to="/signup"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.getStarted') }}
          </NuxtLink>
          <NuxtLink
            to="/login"
            class="text-body lead my-2 text-decoration-none"
            @click.native="collapse"
          >
            {{ $t('navbar.login') }}
          </NuxtLink>
        </div>
      </div>
      <div v-else class="mt-2 input-group d-flex w-100">
        <input
          ref="searchInput"
          v-model="query"
          type="search"
          class="form-control"
          :placeholder="$t('navbar.search')"
          @keyup.enter="searchQuery"
        >
      </div>
    </transition>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import ClickOutside from 'vue-click-outside'

export default {
  name: 'NavbarSmall',
  directives: {
    ClickOutside
  },
  data () {
    return {
      componentKey: 0,
      loading: false,
      show: false,
      query: '',
      icons: {
        notifications: 'notifications_none'
      }
    }
  },
  async fetch () {
    this.refresh = this.timestamp !== null ? new Date().getTime() - this.timestamp > 60 * 1000 * 3 : true
    if (this.refresh === true) {
      await this.getStoreData()
    }
    if (this.isAuthenticated) {
      await this.refreshAccessToken()
      this.fetchNotifications()
    }
  },
  computed: {
    ...mapGetters({
      authorization: 'users/sessions/authorization',
      currentUser: 'users/sessions/user',
      isAuthenticated: 'users/authentication/isAuthenticated',
      notifications: 'notifications/listSorted',
      timestamp: 'timestamp'
    }),
    isScrollTop () {
      return true
    },
    iconMenu () {
      return this.show ? 'clear' : 'menu'
    }
  },
  watch: {
    notifications (oldValue, newValue) {
      if (oldValue.length !== newValue.length) {
        this.icons.notifications = 'notifications'
      }
    }
  },
  beforeMount () {
    window.addEventListener('scroll', this.handleScroll)
  },
  beforeDestroy () {
    window.removeEventListener('scroll', this.handleScroll)
  },
  mounted () {
    this.$store.dispatch('navbarHeight', parseInt(this.$refs.navbar.offsetHeight))
    // window.addEventListener('scroll', this.handleScroll)
  },
  methods: {
    ...mapActions({
      fetchNotifications: 'notifications/list',
      getStoreData: 'getStoreData',
      refreshAccessToken: 'users/sessions/refreshAccessToken'
    }),
    getNotifications () {
      this.icons.notifications = 'notifications_none'
    },
    inputMode () {
      if (this.show === false) {
        this.query = ''
        this.$refs.searchInput.blur()
      }
      return true
    },
    searchQuery () {
      if (this.query.length > 0) {
        this.$router.push({ path: '/search', query: { r: this.query } })
        this.query = ''
      }
    },
    collapseMenu () {
      this.inputMode()
      this.show = !this.show
    },
    collapse () {
      this.inputMode()
      this.show = false
    },
    scroll2Top () {
      this.$fetch()
      if (this.$route.path === '/') {
        const scroll = () => {
          const scrollOptions = {
            top: 0,
            left: 0,
            behavior: 'smooth'
          }
          window.scrollTo(scrollOptions)
        }
        if (window.scrollY > 0) {
          scroll()
        }
        // this.$store.dispatch('recipes/list', {})
      } else {
        this.$router.push({ path: '/' })
      }
    },
    handleScroll (event) {
      this.collapse()
      // Code to be executed when the window is scrolled
      const position = window.scrollY > 0
      // console.log(position)
      if (position) {
        this.$refs.navbar.classList.add('border-bottom')
      } else {
        this.$refs.navbar.classList.remove('border-bottom')
      }
      return position
    },
    logout () {
      this.collapse()
      const message = this.$t('navbar.are_you_sure')

      const options = {
        okText: this.$t('navbar.logout'),
        cancelText: this.$t('navbar.cancel'),
        backdropClose: true, // set to true to close the dialog when clicking outside of the dialog window, i.e. click landing on the mask
        customClass: '' // Custom class to be injected into the parent node for the current dialog instance
      }

      this.$dialog
        .confirm(message, options)
        .then(async (dialog) => {
          // console.log('Clicked on proceed')
          await this.$store.dispatch('users/sessions/logOut', {})
          if (this.$route.path !== '/') {
            this.$router.push({ path: '/' })
          }
        })
        .catch(() => {
          // console.log('Clicked on cancel')
        })
    }
  }
}
</script>
