<script setup>
useHead({
  title: 'Ромка + Лизка',
  link: [
    {
      rel: 'icon',
      type: 'image/svg+xml',
      href: 'data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>❤️</text></svg>'
    }
  ]
})
</script>
<template>
  <LoveCountersAndMagic startDate="2025-02-08T20:00:00" />
</template>
<script>
import LoveCountersAndMagic from '~/components/LoveCountersAndMagic.vue'
export default { components: { LoveCountersAndMagic } }
</script>
