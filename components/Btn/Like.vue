<template>
  <div :class="['d-flex align-items-center text-danger']">
    <div :class="['d-flex align-items-center justify-content-center', { 'flex-column': $device.isMobile }]">
      <div v-if="isAuthenticated" class="d-flex mouse-pointer" @click="likeIt">
        <i :class="['material-icons', liked ? 'text-danger' : 'text-body', $device.isMobile ? 'md-32' : 'md-18']">{{ like }}</i>
      </div>
      <NuxtLink v-else to="/login" class="text-body btn-like">
        <i :class="['material-icons', $device.isMobile ? 'md-32' : 'md-18']">favorite_border</i>
      </NuxtLink>
      <span v-if="!$device.isMobile" :class="['text-muted font-weight-lighter small', { 'ml-1': !$device.isMobile }]">{{ likes }}</span>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'BtnLike',
  props: {
    item: {
      type: Object,
      default: null
    }
  },
  data () {
    return {
      clickable: true,
      likes: this.item.recipe.likes,
      liked: false
    }
  },
  computed: {
    ...mapGetters({
      isAuthenticated: 'users/authentication/isAuthenticated',
      currentUser: 'users/sessions/user'
    }),
    like () {
      return this.liked ? 'favorite' : 'favorite_border'
    }
  },
  mounted () {
    this.isLiked()
  },
  methods: {
    isLiked () {
      if (this.currentUser.likes) {
        this.liked = this.currentUser.likes.findIndex(like => like.recipe_id === this.item.recipe.id) > -1
      } else {
        this.liked = false
      }
    },
    async likeIt () {
      if (this.clickable) {
        this.clickable = false
        if (this.liked) {
          this.likes -= 1
          this.liked = false
          await this.$store.dispatch('recipes/unlike', { user_id: this.currentUser.id, recipe_id: this.item.recipe.id })
        } else {
          this.likes += 1
          this.liked = true
          await this.$store.dispatch('recipes/like', { user_id: this.currentUser.id, recipe_id: this.item.recipe.id })
        }
        this.clickable = true
      }
    }
  }
}
</script>
