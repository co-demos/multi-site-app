<template>
  <div
    v-show="canShowCard"
    :class="`column is-half-tablet is-${colSize}-desktop mb-5`"
    >
    <!-- :class="`column is-${showMore ? 'full' : colSize}`" -->

    <!-- DEBUG -->
    <div 
      v-if="debug"
      class="columns is-multiline"
      >
      <div class="column is-half">
        <p>
          options:
          <br>
          <code>
            <pre>
              {{ options }}
            </pre>
          </code>
        </p>
      </div>
      <div class="column is-half">
        <!-- <p>
          file: {{ file }}
        </p> -->
        <!-- <p>
          content:
          <br>
          <code>
            <pre>
              {{ content }}
            </pre>
          </code>
        </p> -->
      </div>
      <div class="column is-half">
        <p>
          data:
          <br>
          <code>
            <pre>
              {{ data }}
            </pre>
          </code>
        </p>
      </div>
    </div>


    <div 
      v-if="data"
      class="card"
      >

      <!-- TITLE -->
      <header 
        class="card-header"
        @click="openModal()"
        >
        <h2 class="card-header-title is-size-5">
          {{ data[titleKey] }}
        </h2>
      </header>
      
      <!-- COVER -->
      <div 
        v-if="imagesList"
        class="card-image"
        @click="openModal()"
        >
        <b-image
          :src='convertUrl(imagesList[0])'
          :alt='data.name'
          :ratio="imagesRatio"
          :rounded="imagesRounded"
        />
      </div>
      
      <!-- CONTENT -->
      <div 
        class="card-content"
        >

        <!-- TAGS -->
        <div 
          v-if="options['tags-keys']"
          class="content mb-1"
          @click="openModal()"
          >
          <b-taglist
            v-for="(tagKey, idx) in options['tags-keys']"
            :key="`${sectionIndex}-${index}-tag-key-${idx}-${tagKey.key}`"
            class="mb-1"
            >
            <b-tag 
              v-for="tag in data[tagKey.key]"
              :key="`${sectionIndex}-${index}-tag-${idx}-${tag}`"
              rounded
              :type="`is-${tagKey.color}`"
              class="has-text-weight-bold"
              >
              {{ trimString( $translate(tag, itemDict), 25) }}
            </b-tag>
          </b-taglist>
        </div>

        <!-- MINIATURE KEYS -->
        <ul
          v-if="miniaturekKeys"
          @click="openModal()"
          >
          <li 
            v-for="key in miniaturekKeys"
            :key="key"
            >
            <span class="has-text-weight-bold">
              {{ $translate(key, itemDict) }} :
            </span>
            <span>
              {{ data[ key ]}}
            </span>
          </li>
        </ul>

        <hr 
          v-if="options['has-readmore']"
          class="mt-1 mb-5"
        >

        <!-- TEXT -->
        <div 
          class="content is-size-6-touch"
          @click="openModal()"
          >

          <div 
            v-if="options && options['has-readmore']"
            >
            <!-- resume -->
            <VueShowdown
              :markdown="contentSplit.resume + '(...)'"
              :options="showdownOptions"
            />


            <!-- read more -->
            <!-- <VueShowdown
              v-show="showMore"
              :markdown="contentSplit.readMore"
              :options="showdownOptions"
            /> -->
          </div>

          <div
            v-else
            >
            <VueShowdown
              :markdown="content"
              :options="showdownOptions"
            />
          </div>

        </div>

        <!-- button read more -->
        <div 
          class="content"
          >
          <b-button 
            type="is-primary" 
            size="is-small"
            class="mt-3"
            outlined
            expanded
            @click="showMore = !showMore; openModal()"
            >
            <span v-if="!showMore">
              {{ $translate('readmore', defaultDict) }}
              <b-icon icon="plus" size="is-small" class="pl-2"/>
            </span>
            <span v-if="showMore">
              {{ $translate('readless', defaultDict) }}
              <b-icon icon="minus" size="is-small" class="pl-2"/>
            </span>
          </b-button>
        </div>

      </div>

      <!-- FOOTER -->
      <footer
        v-if="options && options['has-socials']"
        class="card-footer"
        >
        <a
          v-for="social in itemSocials"
          :key="social"
          :href="`${ social === 'email' ? 'mailto:' : '' }${ data[social] }`" 
          class="card-footer-item"
          >
          <b-icon :icon="social"/>
        </a>
      </footer>

    </div>


    <!-- MODAL -->
    <b-modal 
      v-model="showModal" 
      :width="'85%'"
      :height="'100%'"
      :has-modal-card="modalConfig['full-screen']"
      :full-screen="modalConfig['full-screen']"
      scroll="keep"
      >
      <DataCardModal
        :modalReady="modalReady"
        :sectionIndex="sectionIndex"
        :index="index"
        :itemData="data"
        :itemContent="content"
        :dataTexts="dataTexts"
        :dataLinks="dataLinks"
        :options="options"
        :itemDict="itemDict"
        :titleKey="titleKey"
        :imagesKey="imagesKey"
        :tagsKeys="tagsKeys"
        :imagesRatio="imagesRatio"
        :imagesList="imagesList"
        :imagesRounded="imagesRounded"
        :fullScreen="modalConfig['full-screen']"
        :debug="false"
        @close="openModal(true); showMore = false"
      />
    </b-modal>

  </div>
</template>


<script>

import matter from 'gray-matter'

import { mapState, mapGetters, mapActions } from 'vuex' 

export default {
  name: 'DataCard',
  components: {
    DataCardModal: () => import(/* webpackChunkName: "DataCardModal" */ '~/components/contents/DataCardModal.vue'),
  },
  props: [
    'sectionIndex',
    'file',
    'options',
    'itemDict',
    'colSize',
    'index',
    'preOpenItem',
    'debug',
  ],
  data() {
    return {
      // images: undefined,
      data: {},
      content: '',
      showMore: false,
      showModal: false,
      modalReady: false,
      socials: [ 'email', 'twitter', 'linkedin', 'github' ],
      defaultDict: {
        readmore: {
          fr: 'Lire plus',
          en: 'Read more'
        },
        readless: {
          fr: 'Lire moins',
          en: 'Read less'
        }
      },
      splittersDict: {
        break: '\n',
        h1: '# ',
        h2: '## ',
        h3: '### ',
        h4: '#### ',
        h5: '##### ',
        h6: '###### ',
      }    }
  },
  computed: {
    ...mapState({
      log: (state) => state.log,
    }),
    ...mapGetters({
      rawRoot : 'getGitRawRoot',
      showdownOptions: 'getShowdownOptions',
      isSelectionActivated: 'data/isSelectionActivated',
      canShowItem: 'data/canShowItem',
    }),
    contentSplit() {
      let contentsArray = [ this.content, '' ]
      const re = /\r\n|\n\r|\n|\r/g
      const authorizedSplitters = Object.keys(this.splittersDict)
      const splitterCode = this.options['readmore-divider'] || 'break'
      const defaultSplitter = this.splittersDict.break
      
      // console.log('-C- DataCard > contentSplit > this.content :', this.content)
      
      if ( authorizedSplitters.includes(splitterCode) ) {

        const splitter = this.splittersDict[splitterCode]
        const contentTrimmed = this.content.startsWith(defaultSplitter) ? this.content.substring(1) : this.content
        // contentsArray = this.content
        contentsArray = contentTrimmed
          .replace(re, defaultSplitter)
          .split(splitter)
          // .filter(c => c !== '')
          // .map( c => {
          //   const str = `${splitter}${c}`
          //   return str
          // })
      }

      return {
        resume: contentsArray[0],
        readMore: contentsArray[1] ? contentsArray.splice(1).join('') : ''
      }
    },
    titleKey() {
      return this.options['title-key'] || 'name'
    },
    imagesKey() {
      return this.options['images-key']
    },
    tagsKeys() {
      return this.options['tags-keys'] || []
    },
    miniaturekKeys() {
      let minKeys = this.options['miniature-keys'] || []
      const tagsKeys = this.tagsKeys.map( t => t.key )
      minKeys = minKeys.filter(  k => !tagsKeys.includes(k) )
      return minKeys
    },
    imagesRatio() {
      return this.options['images-ratio'] || '4by4'
    },
    imagesRounded() {
      return this.options['images-rounded'] 
    },
    imagesList() {
      let imagesArray
      const dataImages = this.data[ this.imagesKey ]
      // console.log('-C- DataCard > imagesList > dataImages :', dataImages)
      if (typeof dataImages === 'string' ) {
        imagesArray = [ dataImages ]
      } else {
        imagesArray = dataImages
      }
      return imagesArray
    },
    itemKeys() {
      return Object.keys(this.data)
    },
    itemSocials() {
      const itemSocials = this.socials.filter( soc => this.itemKeys.includes(soc) )
      return itemSocials
    },
    dataTextsKeys() {
      return this.options['texts-keys']
    },
    dataTexts() {
      const dataTexts = []
      if (this.dataTextsKeys) {
        this.dataTextsKeys.forEach( k => {
          dataTexts.push( { key: k, text: this.data[k] } )
        })
      }
      return dataTexts
    },
    dataLinksKeys() {
      return this.options['links-keys']
    },
    dataLinks() {
      const dataLinks = []
      if (this.dataLinksKeys) {
        this.dataLinksKeys.forEach( k => {
          dataLinks.push( { key: k, text: this.data[k] } )
        })
      }
      return dataLinks
    },
    modalConfig() {
      return this.options['card-modal-config']
    },
    canShowCard() {
      const selectionActivated = this.isSelectionActivated
      const tagsKeys = this.tagsKeys.map( k => k.key )
      if ( selectionActivated ) {
        return this.canShowItem( tagsKeys, this.data)
      } else {
        return true
      }
    }

  },
  watch: {
    data(next) {
      const tagsToAdd = []
      // console.log('-C- DataCard > watch > this.options :', this.options)
      // console.log('-C- DataCard > watch > this.tagsKeys :', this.tagsKeys)
      this.tagsKeys.forEach( tagKey => {
        const tagsArray = next[tagKey.key]
        const tagsObj = {
          key: tagKey.key,
          tags: tagsArray.map( t => {
            return { name: t }
          })
        }
        // console.log('-C- DataCard > watch > tagsObj :', tagsObj)
        tagsToAdd.push(tagsObj)
      })
      // console.log('-C- DataCard > watch > tagsToAdd :', tagsToAdd)
      this.$store.dispatch('data/setAvailableTags', tagsToAdd)
      this.showModal = this.preOpenItem === this.file
    },
    showModal(next) {
      this.changeUrl(!next)
    }
  },
  async mounted() {
    await this.getFileData(this.file)
  },
  methods: {
    ...mapActions({
      setAvailableTags: 'data/setAvailableTags'
    }),
    convertUrl(url) {
      return `${this.rawRoot}${url}`
    },
    async getFileData(url) {
      const urlRaw = this.convertUrl(url)
      // console.log('\n-C- DataCard > getFileData > urlRaw :', urlRaw)
      const req = await this.$axios.get(urlRaw)
      const fileData = matter(req.data)
      // console.log('-C- DataCard > getFileData > fileData :', fileData)
      this.data = fileData.data
      this.content = fileData.content
      this.modalReady = true
      // console.log('-C- DataCard > getFileData > fileData.data :', fileData.data)
      // console.log('-C- DataCard > getFileData > fileData.content :', fileData.content)

    },
    trimString(str, max) {
      const strTrimmed = str.length >= max ? `${str.slice(0, max)}...` : str
      return strTrimmed
    },
    changeUrl(reset) {
      // console.log('\n-C- DataCard > openModal > reset :', reset)
      // console.log('-C- DataCard > openModal > this.file :', this.file)
      // console.log('-C- DataCard > openModal > this.$route :', this.$route)
      const queryItem = reset ? {} : { item: this.file }
      this.$router.push({path: this.$route.path, query: queryItem})
      // const newPath = `${this.$route.path}${query}`
      // console.log('-C- DataCard > openModal > newPath :', newPath)
    },
    openModal(forceClose) {
      this.showModal = forceClose ? false : !this.showModal
    }

  }

}
</script>
