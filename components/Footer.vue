<template>
  <div ref="footer" class="d-print-none py-2 px-3 mt-3 bg-dark text-light d-flex flex-column flex-md-row justify-content-center justify-content-md-between align-items-center">
    <div class="d-flex order-0 order-md-1 align-items-center my-3">
      <NuxtLink
        :to="`/p/${ $t('footer.url.privacyPolicy') }`"
        class="text-light text-decoration-none"
      >
        {{ $t('footer.privacyPolicy') }}
      </NuxtLink>
      <div class="mx-1">
        |
      </div>
      <NuxtLink
        :to="`/p/${ $t('footer.url.sitemap') }`"
        class="text-light text-decoration-none"
      >
        {{ $t('footer.sitemap') }}
      </NuxtLink>
    </div>
    <div class="d-flex order-1 order-md-0 align-items-center">
      <div>{{ $t('footer.madeWith') }}</div>
      <div class="px-1 text-danger">
        <i class="material-icons md-18 d-flex">favorite</i>
      </div>
      <div>
        {{ $t('footer.brand') }} &copy; {{ year }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Footer',
  computed: {
    year () {
      return new Date().getFullYear()
    }
  }
}
</script>
