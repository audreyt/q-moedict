<template>
  <q-page class="word" v-if="data">
    <div v-for = "(b, idx) in bs" :key = "idx">
      <span v-for = "(y, i) in yindiao(w, b, data.h[idx].p)" :key="y.yin + i">
        <h1>{{ y.w }}</h1>
        <span id ="b">
          <span class="yindiao">
            <span class = "yin"> {{y.yin}} </span>
            <span class = "diao"> {{y.diao}} </span>
          </span>
          <span class = "p" v-show="i == 0"> {{y.pin}} </span>
        </span>
      </span>

      <audio id="au" v-if = "data.h[idx]['=']">
        <source :src="'https://203146b5091e8f0aafda-15d41c68795720c6e932125f5ace0c70.ssl.cf1.rackcdn.com/' + data.h[idx]['='] + '.mp3'" type="audio/mp3"/>
        <source :src="'https://203146b5091e8f0aafda-15d41c68795720c6e932125f5ace0c70.ssl.cf1.rackcdn.com/' + data.h[idx]['='] + '.ogg'" type="audio/mp3"/>
      </audio>

      <a id = "play" @click = "play()" v-if = "data.h[idx]['=']">
        <q-icon name="play_arrow" v-if="!playing"/>
        <q-icon name="pause" v-else/>
      </a>

      <span v-if = "data.r">
        <span class="radical">{{ p(data.r)[0] }}</span> + {{ data.n }} = {{ data.c }}
      </span>
      <a class ="star" v-if = "stars.indexOf(w) == -1" @click = "star(w)">
        <q-icon name="star_outline" />
      </a>
      <a class ="star"  v-else @click="unstar(w)">
        <q-icon name="star" />
      </a>
      <div v-if = "data">
        <ol>
          <li v-for = "d in data.h[idx].d" :key = "d.f">
            <span v-if = "d.type">
              <span class="type">{{ p(d.type)[0] }}</span>：
            </span>
            <span class="def" v-if = "d.f">
              <router-link v-for = "(r, idx) in p(d.f)" :to = "'/w/' + r" :key = "r+idx" :disabled="dis(r)" :event="!dis(r) ? 'click' : ''">{{ r }}</router-link>
            </span>
            <div v-if = "d.e">
              <div v-for = "e in d.e" :key="e">
                <router-link v-for = "(r, idx) in p(e)" :to = "'/w/' + r" :key = "r+idx" :disabled="dis(r)"  :event="!dis(r) ? 'click' : ''">{{ r }}</router-link>
              </div>
            </div>
            <br/>
            <ol>
              <li v-for = "q in d.q" :key="q">
                <router-link v-for = "(r, idx) in p(q)" :to = "'/w/' + r" :key = "r+idx" :disabled="dis(r)"  :event="!dis(r) ? 'click' : ''">{{ r }}</router-link>
              </li>
            </ol>
            <span class="antonyms" v-if = "d.a">
              <span class="type">反</span>
                <router-link v-for = "(r, idx) in p(d.a)" :to = "'/w/' + r" :key = "r+idx" :disabled="dis(r)"  :event="!dis(r) ? 'click' : ''">{{ r }}</router-link>
            </span>
            <br/>
          </li>
        </ol>
      </div>
      <span class="antonyms" v-if = "data.English">
        <span class="type">英</span> {{data.English}}<br/>
      </span>
      <span class="antonyms" v-if = "data.Deutsch">
        <span class="type">德</span> {{data.Deutsch}}<br/>
      </span>
      <span class="antonyms" v-if = "data.francais">
        <span class="type">法</span> {{data.francais}}<br/>
      </span>
    </div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      w: '',
      bs: [],
      data: null,
      playing: false
    }
  },
  props: ['stars'],
  meta () {
    return {
      // this accesses the "title" property in your Vue "data";
      // whenever "title" prop changes, your meta will automatically update
      title: this.w + ' - 萌典'
    }
  },
  mounted () {
    this.w = this.$route.params.id
    this.$axios.get('https://www.moedict.tw/a/' + this.w + '.json')
      .then((response) => {
        this.data = response.data
        console.log(this.data)
        this.bs = this.data.h.map((o) => { return o.b })
        this.playing = false
      })
  },
  methods: {
    play () {
      if (!this.playing) {
        document.getElementById('au').load()
        document.getElementById('au').play()
        this.playing = true
      } else {
        document.getElementById('au').pause()
        this.playing = false
      }
    },
    star (w) {
      var arr = this.$q.localStorage.getItem('words')
      if (!arr) { arr = [] }
      arr = arr.filter((x) => { return x !== this.w })
      arr.push(w)
      this.$q.localStorage.set('words', arr)
      this.$emit('updateStars')
    },
    unstar (w) {
      var arr = this.$q.localStorage.getItem('words')
      if (!arr) { arr = [] }
      arr = arr.filter((x) => { return x !== this.w })
      this.$q.localStorage.set('words', arr)
      this.$emit('updateStars')
    },
    yindiao (w, b, p) {
      var word = w
      var arr = ('' + b).split('　')
      return arr.map((k, idx) => {
        k = k.replace(/（.+）/g, '').replace('ㄧ', '─')
        var obj = {
          w: word[idx],
          yin: k.substr(0, k.length - 1),
          diao: k.substr(k.length - 1, k.length),
          pin: p
        }

        if (obj.diao !== 'ˋ' && obj.diao !== 'ˊ' && obj.diao !== 'ˇ' && obj.diao !== 'ˊ') {
          obj.yin = obj.yin + obj.diao
          obj.diao = ''
        }

        return obj
      })
    },
    dis (w) {
      return w.match(/(⚋|⚊|☰|☱|☲|☳|☴|☵|☶|☷|灾|从|0|1|2|3|4|5|6|7|8|9|：|《|》|〈|〉|．|、|。|；|「|」|『|』|（|）|\(|\)|，)/)
    },
    p (s) {
      var a = s
        .replace(/{\[8ebc\]}/g, '⚋')
        .replace(/{\[8ebd\]}/g, '⚊')
        .replace(/{\[8e79\]}/g, '☰')
        .replace(/{\[8e7b\]}/g, '☱')
        .replace(/{\[8e7c\]}/g, '☲')
        .replace(/{\[8e7e\]}/g, '☳')
        .replace(/{\[8e7d\]}/g, '☴')
        .replace(/{\[8ea1\]}/g, '☵')
        .replace(/{\[8ea2\]}/g, '☶')
        .replace(/{\[8e7a\]}/g, '☷')
        .replace(/{\[9264\]}/g, '灾')
        .replace(/{\[9064\]}/g, '从')
      var arr = [...a.matchAll(/(⚋|⚊|☰|☱|☲|☳|☴|☵|☶|☷|灾|从|0|1|2|3|4|5|6|7|8|9|：|《|》|〈|〉|．|、|。|；|「|」|『|』|（|）|\(|\)|，|`(.+?)~)/g)].map((o) => {
        const w = o.filter((k) => { return k })
        // console.log(w)
        return w[w.length - 1]
      })
      return arr
    }
  },
  watch: {
    $route (to, from) {
      this.w = this.$route.params.id
      this.$axios.get('https://www.moedict.tw/a/' + this.w + '.json')
        .then((response) => {
          this.data = response.data
          console.log(this.data)
          this.bs = this.data.h.map((o) => { return o.b })
          this.playing = false
        })
      this.$forceUpdate()
    }
  }
}
</script>

<style type="text/css" scoped="">

  .word {
    padding: 0 1em;
  }

  h1 {
    font-size: 64px;
    display: inline;
    margin-right: .3em;
  }

  #b {
    position: relative;
    right: .5em;
    top: .9em;
    display: inline-block;
    font-size: 24px;
    line-height: 100%;
    width: 1em;
    height: 4em;
    overflow: visible;
  }

  .diao {
    position: relative;
  }

  a {
    cursor: pointer;
    text-decoration: none;
  }

  .r {
    text-decoration: none;
  }

  .r::after {
  }

  .type, .radical {
    background-color: rgb(225,100,100);
    border-radius: 2px;
    margin-right: .2em;
    padding: 0 .1em;
  }

  .star {
    font-size: 24px;
    color: gold;
  }

  .yindiao {
    display: inline-flex;
    align-items: center;
    position: relative;
    top: -2.2em;
    color: gray;
  }

  .yin {
    line-height: 1em;
  }

  #play {
    color: #c33;
    margin-right: 1em;
    font-size: 40px;
  }

  .p {
    position: absolute;
    bottom: 2.2em;
    left: -2.5em;
    color: gray;
    width: 10em;
  }

</style>
