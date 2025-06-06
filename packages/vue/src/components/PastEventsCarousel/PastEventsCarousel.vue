<template>
  <MixinCarousel
    v-if="hasContent"
    heading="Past Events"
    variant="cards"
    link="/events/?page=1&event_status=Past+events&sortBy=eventStartDateLatest"
    link-title="View all past events"
    indent="col-1"
    v-bind="$attrs"
  >
    <BlockLinkCard
      v-for="(page, index) in pastEvents"
      :key="index"
      :url="page.url"
      :title="page.title"
      :thumbnail-image="page.image"
      :start-date="page.startDate"
      :end-date="page.endDate"
      class="swiper-slide"
    />
  </MixinCarousel>
</template>
<script lang="ts">
// relative link to view past events assumes that this component will only be used on an events index page.
// @ts-nocheck
// @ts-expect-error Elastic search response is not fully typed
import { defineComponent } from 'vue'
import type { ElasticSearchPage } from '../../interfaces'
import MixinCarousel from './../MixinCarousel/MixinCarousel.vue'
import BlockLinkCard from './../BlockLinkCard/BlockLinkCard.vue'

export default defineComponent({
  name: 'PastEventsCarousel',
  components: {
    MixinCarousel,
    BlockLinkCard
  },
  props: {
    data: {
      type: Object,
      required: true
    }
  },
  computed: {
    hasContent() {
      if (this.data?.length) {
        return true
      }
      return false
    },
    pastEvents(): ElasticSearchPage[] {
      return this.data
        .filter((page: ElasticSearchPage) => {
          return 'url' in page._source
        })
        .map((page: ElasticSearchPage) => {
          // helpers
          const handle = this.parseType(page._source.content_type[0])
          const image = page._source[handle + '__thumbnail_image']
          page.url = page._source.url
          page.title = page._source.title
          page.startDate = page._source[handle + '__start_datetime']
          page.endDate = page._source[handle + '__end_datetime']
          if (image) {
            page.image = {
              src: {
                url: image.thumbnail_image
              }
            }
          }
          return page
        })
    }
  },
  methods: {
    parseType(type: string): string {
      return type.toLowerCase().replace('.', '_')
    }
  }
})
</script>
