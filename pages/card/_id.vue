<template>
  <div></div>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'
import { CardRoot } from '~/plugins/firebase'

let id = ''
let photoURL = ''
let title = ''
let username = ''

@Component
export default class EVENT_ID extends Vue {
  id = ''
  photoURL = ''
  photoName = ''
  title = ''
  username = ''

  async asyncData({ route }: any) {
    const eventDoc = await CardRoot.doc(route.params.id).get()
    const eventData = eventDoc.data()
    if (eventData !== undefined) {
      id = eventData.id
      photoURL = eventData.photoURL
      title = eventData.title
      username = eventData.username
    }
  }

  head() {
    return {
      meta: [
        { hid: 'og:type', property: 'og:type', content: 'article' },
        {
          hid: 'og:image',
          property: 'og:image',
          content: photoURL,
        },
        {
          hid: 'og:title',
          property: 'og:title',
          content: title + 'で打線組んでみた',
        },
        {
          hid: 'og:description',
          property: 'og:description',
          content: '',
        },
        {
          name: 'twitter:card',
          content: 'summary_large_image',
        },
        {
          name: 'twitter:image',
          content: photoURL,
        },
      ],
    }
  }

  mounted() {
    this.$router.push('/')
  }

  get tweetURL(): string {
    return (
      'http://twitter.com/share?related=@twibo_app&hashtags=ツイ募&text=' +
      this.title +
      'のメンバーを募集中です！' +
      '&url=' +
      process.env.BASE_URL +
      '/' +
      this.$route.fullPath
    )
  }
}
</script>

<style lang="scss" scoped>
/* 画像のアスペクト比は固定なので、あらかじめ高さをCSSで調整 */
.preview-wrapper {
  position: relative;
  width: 100%;
  height: auto;
}

.preview-wrapper {
  &:before {
    content: '';
    display: block;
    padding-top: 52.5%; /* 630 / 1200 x 100 */
  }
}

.preview-content {
  position: absolute;
  top: 0;
  left: 0;
}

.btn-title {
  color: white;
  font-size: 1rem;
  font-weight: bold;
}
</style>
