<template>
  <div class="container pt-5">
    <div v-if="$fetchState.pending">
      <Loading />
    </div>
    <div v-else-if="$fetchState.error">
      <NotFound />
    </div>
    <div v-else>
      <SocialHead
        :title="socialMetaData.title"
        :description="socialMetaData.description"
        :image="socialMetaData.image"
      />
      <div v-if="!$device.isMobile && currentUser.admin" class="d-flex justify-content-center align-items-center my-3">
        <NuxtLink :to="`/p/${ $route.params.slug }/edit`" class="text-body text-capitalize text-decoration-none">
          {{ $t('pages.edit') }}
        </NuxtLink>
      </div>
      <div>
        <Markdown :source="item.content" />
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'Page',
  data () {
    return {
      item: {
        content: '',
        title: ''
      }
    }
  },
  async fetch () {
    await this.fetchItems()
    this.item = this.page(this.$route.params.slug)
  },
  computed: {
    ...mapGetters({
      page: 'pages/filter',
      currentUser: 'users/sessions/user'
    }),
    socialMetaData () {
      return {
        title: this.item.title,
        description: 'Partagez vos recettes dès maintenant en toute simplicité',
        image: 'https://media.cuisinierrebelle.com/images/cr_icon_1200x1200.jpg'
      }
    }
  },
  methods: {
    ...mapActions({
      fetchItems: 'pages/get'
    })
  }
}
</script>
