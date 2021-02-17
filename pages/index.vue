<template>
  <div>
    <script
      type="text/javascript"
      async
      src="https://platform.twitter.com/widgets.js"
    ></script>

    <v-card>
      <v-container fluid>
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="username"
              label="あなたのTwitterアカウント名(任意)"
              prefix="@"
            ></v-text-field>
          </v-col>

          <v-col cols="12">
            <v-text-field v-model="title" label="打線名"></v-text-field>
            <div align="right">
              <span class="title bold" style="font-weight: bold">
                で打線組んだ
              </span>
            </div>
          </v-col>
          <v-col cols="12">
            <client-only>
              <div id="preview-wrapper" class="preview-wrapper">
                <v-stage
                  ref="stage"
                  :config="configKonva"
                  class="preview-content"
                  :style="style"
                >
                  <v-layer>
                    <v-image :config="configBg"></v-image>
                    <v-text :config="configContent1"></v-text>
                    <v-text :config="configContent2"></v-text>
                    <v-text :config="configContent3"></v-text>
                    <v-text :config="configContent4"></v-text>
                    <v-text :config="configContent5"></v-text>
                    <v-text :config="configContent6"></v-text>
                    <v-text :config="configContent7"></v-text>
                    <v-text :config="configContent8"></v-text>
                    <v-text :config="configContent9"></v-text>
                  </v-layer>

                  <v-layer>
                    <v-text :config="configContentPosition1"></v-text>
                    <v-text :config="configContentPosition2"></v-text>
                    <v-text :config="configContentPosition3"></v-text>
                    <v-text :config="configContentPosition4"></v-text>
                    <v-text :config="configContentPosition5"></v-text>
                    <v-text :config="configContentPosition6"></v-text>
                    <v-text :config="configContentPosition7"></v-text>
                    <v-text :config="configContentPosition8"></v-text>
                    <v-text :config="configContentPosition9"></v-text>
                  </v-layer>
                  <v-layer>
                    <v-text :config="configContentNumber1"></v-text>

                    <v-text :config="configContentNumber2"></v-text>

                    <v-text :config="configContentNumber3"></v-text>

                    <v-text :config="configContentNumber4"></v-text>

                    <v-text :config="configContentNumber5"></v-text>

                    <v-text :config="configContentNumber6"></v-text>

                    <v-text :config="configContentNumber7"></v-text>

                    <v-text :config="configContentNumber8"></v-text>

                    <v-text :config="configContentNumber9"></v-text>
                  </v-layer>
                </v-stage>
              </div>
            </client-only>
          </v-col>
        </v-row>
        <v-row
          v-for="(content, i) in contents"
          :key="i"
          style="margin-bottom: -30px"
          justify="center"
        >
          <v-col cols="3" sm="3" md="2" style="padding-right: 0px">
            <v-select
              v-model="content.position"
              :items="positions"
              dense
              label="位置"
            ></v-select>
          </v-col>
          <v-col cols="9" sm="9" md="10">
            <v-text-field
              v-model="content.content"
              outlined
              dense
              :label="i + 1 + '番'"
            >
            </v-text-field>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" class="d-flex justify-center">
            <v-btn
              target="_blank"
              color="#00ACEE"
              @click.stop="handleCreateCard"
            >
              <v-icon size="25" color="#fff" left>mdi-twitter</v-icon>
              <span class="btn-title">投稿する</span>
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-card>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'
import firebase from 'firebase/app'
import { CardRoot } from '~/plugins/firebase.ts'

const IMAGE_WIDTH = 1196
const IMAGE_HEIGHT = 630
const USER_IMAGE_SIZE = 110
const USER_IMAGE_X = 51
const USER_IMAGE_Y = 466
const CONTENT_X = 190
const CONTENT_POSITION_X = 120
const CONTENT_NUMBER_X = 65

const TEXT_Y_ADJUST = -3

const TEXT_Y = [
  30 + TEXT_Y_ADJUST,
  90 + TEXT_Y_ADJUST,
  150 + TEXT_Y_ADJUST,
  210 + TEXT_Y_ADJUST,
  270 + TEXT_Y_ADJUST,
  330 + TEXT_Y_ADJUST,
  390 + TEXT_Y_ADJUST,
  450 + TEXT_Y_ADJUST,
  510 + TEXT_Y_ADJUST,
]

@Component
export default class Page extends Vue {
  private configKonva = { width: IMAGE_WIDTH, height: IMAGE_HEIGHT }
  private configBg: { image: HTMLImageElement | null } = { image: null }
  private configUserImg: any | null = null
  private scale: number = 0

  positions = ['投', '捕', '一', '二', '三', '遊', '左', '中', '右']

  contents = [
    { position: '中', content: 'Ruby' },
    { position: '二', content: 'Python' },
    { position: '遊', content: 'C#' },
    { position: '左', content: 'JavaScript' },
    { position: '一', content: 'Java' },
    { position: '三', content: 'Swift' },
    { position: '捕', content: 'C' },
    { position: '右', content: 'Dart' },
    { position: '投', content: 'C++' },
  ]

  username = ''

  title = ''

  async mounted() {
    await this.setupBg()
    await this.setupUserImage()

    // 初回のスケール計算処理を走らせる
    this.$nextTick(() => this.handleResize())
    // イベントリスナーにスケール計算処理を登録
    window.addEventListener('resize', this.handleResize)
  }

  async handleCreateCard() {
    const cardDocRef = CardRoot.doc()

    const currentDate: Date = new Date()

    let photoURL = ''

    try {
      await this.uploadImage(cardDocRef.id)
        .then((ImageValue: any) => {
          photoURL = ImageValue.photoURL
        })
        .catch((error) => {
          throw new Error(error.message)
        })

      const newCard: any = {
        id: cardDocRef.id,
        photoURL,
        username: this.username,
        title: this.title,
        createdAt: currentDate,
        updatedAt: currentDate,
      }

      await cardDocRef
        .set(newCard)
        .catch()
        .then(() => {
          if (
            navigator.userAgent.indexOf(
              'iPhone' || 'iPod' || 'iPad' || 'Andoroid'
            ) > 0
          ) {
            location.href =
              'https://twitter.com/intent/tweet?related=@twibo_app&hashtags=打線メーカー&text=' +
              this.title +
              'で打線組んだ' +
              '&url=' +
              'https://dasen.me' +
              '/card/' +
              cardDocRef.id
          } else {
            window.open(
              'https://twitter.com/intent/tweet?related=@twibo_app&hashtags=打線メーカー&text=' +
                this.title +
                'で打線組んだ' +
                '&url=' +
                'https://dasen.me' +
                '/card/' +
                cardDocRef.id,
              '_brank'
            )
          }
        })
        .catch((error: string) => {
          throw new Error(error)
        })
    } catch (error) {
      console.log(error)
      alert('打線の作成に失敗しました。')
    }
  }

  private get configUserImageGroup() {
    return {
      // 複雑な切り取り処理: CanvasRenderingContext2Dのarcを使う
      clipFunc: (ctx: any) => {
        const r = USER_IMAGE_SIZE / 2
        ctx.arc(USER_IMAGE_X + r, USER_IMAGE_Y + r, r, 0, Math.PI * 2, false)
      },
      // ドラッグを無効にする
      draggable: false,
    }
  }
  /*
  private get configTitle() {
    return {
      // 表示する文字
      text: this.title,
      x: 200,
      y: 70,
      width: IMAGE_WIDTH - 400,
      fill: '#204495', // 文字の色
      fontSize: 65, // フォントサイズ
      fontFamily: 'Noto Sans JP', // フォントの種類
      fontStyle: 'bold', // フォントのスタイル
      align: 'center', // 揃え位置(中央揃え)
      // 折り返しなし(wrap)を指定すると、省略(ellipsis)を設定可能に
      wrap: 'none',
      ellipsis: true,
    }
  }
  */

  private get configContent1() {
    return {
      text: this.contents[0].content,
      x: CONTENT_X,
      y: TEXT_Y[0],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent2() {
    return {
      text: this.contents[1].content,
      x: CONTENT_X,
      y: TEXT_Y[1],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent3() {
    return {
      text: this.contents[2].content,
      x: CONTENT_X,
      y: TEXT_Y[2],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent4() {
    return {
      text: this.contents[3].content,
      x: CONTENT_X,
      y: TEXT_Y[3],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent5() {
    return {
      text: this.contents[4].content,
      x: CONTENT_X,
      y: TEXT_Y[4],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent6() {
    return {
      text: this.contents[5].content,
      x: CONTENT_X,
      y: TEXT_Y[5],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent7() {
    return {
      text: this.contents[6].content,
      x: CONTENT_X,
      y: TEXT_Y[6],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent8() {
    return {
      text: this.contents[7].content,
      x: CONTENT_X,
      y: TEXT_Y[7],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContent9() {
    return {
      text: this.contents[8].content,
      x: CONTENT_X,
      y: TEXT_Y[8],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  // ユーザ画像の読み込み処理: 本の画像とかとだいたい同じ

  private async setupUserImage() {
    const userImg =
      'https://pbs.twimg.com/profile_images/1302171026931949569/IB5B3x89_normal.jpg'

    const image = await this.getImage(userImg)

    this.configUserImg = {
      image,
      x: USER_IMAGE_X,
      y: USER_IMAGE_Y,
      width: USER_IMAGE_SIZE,
      height: USER_IMAGE_SIZE,
    }
  }

  beforeDestroy() {
    // Destroy前に解放
    window.removeEventListener('resize', this.handleResize)
  }

  private get style() {
    // transformのscaleを使って、サイズ調整
    return {
      transform: `scale(${this.scale})`,
      '-webkit-transform': `scale(${this.scale})`,
      'transform-origin': '0 0',
    }
  }

  // スケール計算用の処理: 対象IDのサイズを取得して、scaleを計算
  private handleResize() {
    const elm = document.getElementById('preview-wrapper')
    if (!elm) return
    const rect = elm.getBoundingClientRect()

    this.scale = rect.width / IMAGE_WIDTH
  }

  private async setupBg() {
    const image = await this.getImage('/dasen3.png')
    this.configBg.image = image
  }

  private getImage(src: string): Promise<HTMLImageElement> {
    // eslint-disable-next-line @typescript-eslint/no-unused-vars
    return new Promise((resolve, reject) => {
      const image = new window.Image()
      image.setAttribute('crossOrigin', 'anonymous')
      image.onload = () => resolve(image)
      image.src = src
    })
  }

  uploadImage(fileName: string) {
    const thisClass = this
    return new Promise((resolve, reject) => {
      // stageの取得: 大きいサイズを取得するときは、'pixelRatio'で倍率を指定
      const stage = (thisClass.$refs.stage as any).getStage({ pixelRatio: 1 })

      // stageからDataURLを取得
      const dataURL = stage.toDataURL()
      const byteString = atob(dataURL.split(',')[1])
      const mimeType = dataURL.match(/(:)([a-z\/]+)(;)/)[2]
      const content = new Uint8Array(byteString.length)

      for (let i = 0; byteString.length > i; i++) {
        content[i] = byteString.charCodeAt(i)
      }
      const blob = new Blob([content], {
        type: mimeType,
      })

      const ref = firebase.storage().ref().child('card').child(fileName)

      ref
        .put(blob, {
          customMetadata: {
            owner: '',
          },
        })
        .then(() => {
          ref.getDownloadURL().then((url) => {
            resolve({ photoURL: url, photoName: ref.name })
          })
        })
        .catch((err) => {
          reject(err)
        })
    })
  }

  private get configContentPosition1() {
    return {
      text: this.contents[0].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[0],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition2() {
    return {
      text: this.contents[1].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[1],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition3() {
    return {
      text: this.contents[2].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[2],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition4() {
    return {
      text: this.contents[3].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[3],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition5() {
    return {
      text: this.contents[4].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[4],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition6() {
    return {
      text: this.contents[5].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[5],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition7() {
    return {
      text: this.contents[6].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[6],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition8() {
    return {
      text: this.contents[7].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[7],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentPosition9() {
    return {
      text: this.contents[8].position,
      x: CONTENT_POSITION_X,
      y: TEXT_Y[8],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber1() {
    return {
      text: 1,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[0],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber2() {
    return {
      text: 2,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[1],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber3() {
    return {
      text: 3,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[2],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber4() {
    return {
      text: 4,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[3],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber5() {
    return {
      text: 5,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[4],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber6() {
    return {
      text: 6,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[5],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber7() {
    return {
      text: 7,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[6],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber8() {
    return {
      text: 8,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[7],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
  }

  private get configContentNumber9() {
    return {
      text: 9,
      x: CONTENT_NUMBER_X,
      y: TEXT_Y[8],
      width: 900,
      height: 250,
      fill: '#333333',
      fontStyle: 'bold',
      fontSize: 40,
      fontFamily: 'Noto Sans JP',
      // 行の高さ
      lineHeight: 1.2,
    }
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
