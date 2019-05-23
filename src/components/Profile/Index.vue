<template>
  <div class="page">
    <h1> 
      <router-link tag="i" class="fa fa-arrow-left" to="/"></router-link>
      Welcome {{userStore.currentUser.firstname}} 
    </h1>
    <div class="searchUtil">
    <input class="black search" type="text" placeholder="🔍 Search by title..." v-model=searchStr @change=searchTextChanged>
    </div>
    <CodeList :codes=codes></CodeList>
    <button type="button" class="btn btn-sm btn-run" @click="loadMore()" 
        :class="{ loading : disabled }" :disabled="loading" v-if="haveMoreCodes">
      <span v-if="loading">Loading</span>
      <span v-else>Load More</span>
    </button>
  </div>
</template>


<script>
import { httpGet } from '@/utils/api'
import { mapState } from 'vuex'

import CodeList from './CodeList.vue'

export default {
  name: 'profile',
  components: {
    CodeList
  },
  data () {
    return {
      searchStr: '',
      codes: [],
      currentOffset: 0,
      loading: false,
      codeCount: 0
    }
  },
  computed: {
    haveMoreCodes: function() {
      return this.currentOffset + 20 < codeCount
    },
    ...mapState({
      userStore: 'user'
    })
  },
  async created () {
    this.fetchCodes()
  },
  methods: {
    async fetchCodes (title = '', offset = 0, limit) {
      const { data } = await httpGet('/code', {
        filter: {
          title: {
            $iLike: `%${title}%`
          }
        },
        offset,
        limit
      })
      this.codes = data.codes
      this.codeCount = data.count
    },
    async loadMore () {
      try {
        this.loading = true
        const title = this.searchStr
        const offset = this.currentOffset + 20
        const { data } = await httpGet('/code', {
          filter: {
            title: {
              $iLike: `%${title}%`
            }
          },
          offset
        })
        this.codes.concat(data.codes)
        this.offset = offset
      } catch (err) {
        console.log(err)
      }
      this.loading = false
    },
    searchTextChanged (e) {
      this.fetchCodes(this.searchStr)
    },
    goBack () {
      this.$router.push({ name: 'root'})
    }
  }
  
}
</script>

<style scoped>
  .page {
    height: 100vh;
    padding: 1% 10%;
    overflow: auto;
  }
  h1 {
    color: whitesmoke;
  }
  .searchUtil {
    display: flex;
    justify-content: center;
    font-size: 35px;
  }
  input.search {
    width: 50%;
  }
  i {
    cursor: pointer;
  }
</style>