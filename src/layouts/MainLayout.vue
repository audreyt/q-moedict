<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          <input id="s" type="search" name="s" v-model="myKey" list="words" placeholder="輸入字詞" @keydown.enter="$router.push('/w/' + myKey)"/>
          <datalist id ="words">
            <option v-for = "d in has(data, myKey).slice(0,n)" :key="d" :value="d"></option>
            }
          </datalist>
          <button @click="$router.push('/w/' + myKey)">查詢</button>
        </q-toolbar-title>
        <div class="gcse-search"></div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-1"
    >
    <q-infinite-scroll @load="onLoad" :offset="250"
      :scroll-target="$refs.scrollTargetRef">
      <q-list bordered>
        <q-item>
          <b>已加星號</b>
        </q-item>
        <q-item clickable v-for = "k in stars" :to = "'/w/' + k" :key="k">
          {{k}}
        </q-item>
      </q-list>
    </q-infinite-scroll>
    </q-drawer>

    <q-page-container>
      <router-view @updateStars = "updateStars()" :stars="stars"/>
    </q-page-container>
  </q-layout>
</template>

<script>

export default {
  name: 'MainLayout',
  data () {
    return {
      myKey: '',
      data: [],
      stars: [],
      leftDrawerOpen: false,
      n: 100
    }
  },
  methods: {
    deep () {
      window.IS_GOOGLE_AFS_IFRAME_ = true
      const cx = '007966820757635393756:sasf0rnevk4'
      var gcse = document.createElement('script')
      gcse.type = 'text/javascript'
      gcse.async = true
      gcse.src = '//www.google.com/cse/cse.js?cx=' + cx
      var s = document.getElementsByTagName('script')[0]
      s.parentNode.insertBefore(gcse, s)
      setInterval(function () {
        var e = document.getElementById('gsc-i-id1')
        if (e) {
          e.setAttribute('placeholder', '全站搜詢')
        }
      }, 500)
    },
    onLoad () {
      this.n += 50
    },
    has (data, k) {
      // console.log(data)
      return data.filter((x) => { return x.indexOf(k) > -1 })
    },
    updateStars () {
      this.stars = this.$q.localStorage.getItem('words') || []
    }
  },
  mounted () {
    var vm = this
    this.$axios.get('https://www.moedict.tw/a/index.json')
      .then((response) => {
        this.data = response.data
      })
    this.stars = this.$q.localStorage.getItem('words') || []
    this.deep()
    setInterval(function () {
      var list = document.getElementsByClassName('gs-title')
      for (var i = 0; i < list.length; i++) {
        const e = list[i]
        if (e.getAttribute('href')) {
          var l = e.getAttribute('href')
          l = ('' + l).replace('https://www.moedict.tw/', '')
          e.removeAttribute('href')
          e.removeAttribute('data-cturl')
          e.removeAttribute('data-ctorig')
          e.setAttribute('data-h', l)
          e.setAttribute('target', 'self')
          e.addEventListener('click', function () {
            console.log(this.getAttribute('data-h'))
            vm.$router.push('/w/' + this.getAttribute('data-h'))
            var x = document.getElementsByClassName('gsc-modal-background-image gsc-modal-background-image-visible')
            x[0].click()
            vm.$forceUpdate()
          })
        }
      }
    }, 1000)
  }
}
</script>

<style type="text/css">
  #s {
    width: 140px !important;
  }
</style>
