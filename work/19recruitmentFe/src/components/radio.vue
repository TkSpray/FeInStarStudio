<template>
  <div class="radio">
    <span class="headline" v-html="trimstr(title)"></span>
    <div class="rad" ref="radioref">
      <div v-for="(opt, indexr) in options" :key="indexr" class="optall">
        <input
          type="radio"
          class="radio1"
          :id="radio(indexr)"
          :value="options[indexr]"
          v-model="radiodata"
          @change="sendradio"
          :disabled="isread"
        />
        <label :for="radio(indexr)" class="radio2">{{ options[indexr] }}</label>
      </div>
    </div>
    <div class="ctrlBox" v-if="$route.path == '/adminindex/ctrlques'">
      <p>出题人：{{ author }}</p>
      <input type="button" value="添加" @click="toAdd" />
      <!-- <select name="frontOrBack"
              id="frontOrBack">
        <option value="于此题后">于此题后</option>
        <option value="于此题前">于此题前</option>
      </select> -->
      <input type="button" value="删除" @click="delBoxFlag = true" />
      <input type="button" value="修改" @click="toChange" />
      <div class="delBox" v-if="delBoxFlag">
        <p>是否删除</p>
        <img src="../assets/删除@3x.svg" />
        <input type="button" value="确认" @click="delConfirm" />
        <input type="button" value="取消" @click="delBoxFlag = false" />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      delBoxFlag: false,
      groups: this.$route.query.groups,
      list1: [],
      radiodata: '',
      isread: false,
      thistime: ''
    }
  },
  props: {
    options: {
      type: Array,
      default: () => []
    },
    ID: {
      type: Number,
      default: 0
    },
    title: {
      type: String,
      default: ''
    },
    answer: {
      type: String,
      default: ''
    },
    index: {
      type: Number,
      default: 0
    },
    author: {
      type: String,
      default: ''
    }
  },
  methods: {
    radio: function(index2) {
      if (this.$route.path === 'answer') {
        return 'radio' + this.index + index2
      } else {
        return 'radio' + this.ID + index2
      }
    },
    trimstr: function(str) {
      if (this.$route.path !== '/adminindex/ctrlques') {
        let strindex = String(this.index + 1)
        let strtrim = '(单选题) '
        let head = strindex + '.' + strtrim
        let strtrim1 = str.replace(/\n|\r\n/g, '<br/>')
        let strtrim2 = strtrim1.replace(/\s/g, '&nbsp')
        let tempstr = strtrim2.replace(/img&nbsp/g, 'img ')
        let strtrim3 = head.concat(tempstr)
        return strtrim3
      } else {
        let strtrim = '(单选题) '
        let strtrim1 = str.replace(/\n|\r\n/g, '<br/>')
        let strtrim2 = strtrim1.replace(/\s/g, '&nbsp')
        let tempstr = strtrim2.replace(/img&nbsp/g, 'img ')
        let strtrim3 = strtrim.concat(tempstr)
        return strtrim3
      }
    },
    sendradio: function(value) {
      if (
        this.$route.path !== '/adminindex/ctrlques' &&
        this.$route.path !== '/marking'
      ) {
        this.$axios({
          method: 'post',
          url: '/user/exam/answer',
          data: {
            ID: this.ID,
            answer: this.radiodata
          }
        })
          .then(res => {
            if (res.data.code === 0) {
              let mytime = new Date()
              this.thistime = ''
              if (mytime.getHours() < 10) {
                this.thistime += '0'
              }
              this.thistime += mytime.getHours() + ':'
              if (mytime.getMinutes() < 10) {
                this.thistime += '0'
              }
              this.thistime += mytime.getMinutes() + ':'
              if (mytime.getSeconds() < 10) {
                this.thistime += '0'
              }
              this.thistime += mytime.getSeconds()
              this.thistime += ' ' + '自动保存成功'
              this.$emit('trantime', this.thistime)
              this.$message({
                message: this.thistime,
                type: 'success'
              })
            } else {
              alert('答案上传失败')
            }
          })
          .catch(err => {
            if (err) {
              alert('答案上传失败')
              this.$emit('tranalert')
            }
          })
      }
    },
    toAdd() {
      this.$router.push({
        path: '/adminindex/add',
        query: {
          groups: parseInt(this.groups)
        }
      })
    },
    toChange() {
      this.$router.push({
        path: '/adminindex/add',
        query: {
          ID: this.ID,
          groups: parseInt(this.groups)
        }
      })
    },
    delConfirm() {
      this.delBoxFlag = false
      this.$axios({
        method: 'post',
        url: '/control/question/del',
        data: {
          ID: this.ID
        }
      })
        .then(result => {
          console.log(result)
          this.$router.go(0)
        })
        .catch(err => {
          console.log(err)
        })
    }
  },
  mounted() {
    if (this.answer !== '') {
      this.radiodata = this.answer
    }
    if (this.$route.path === '/adminindex/ctrlques') {
      this.isread = true
      console.log(this.isread)
      let answers = []
      for (let i = 0; i < this.options.length; i++) {
        answers.push(this.options[i].content)
        if (this.options[i].answer === 1) {
          this.radiodata = this.options[i].content
        }
      }
      for (let i = 0; i < this.options.length; i++) {
        this.options.splice(i, 1, answers[i])
      }
      // console.log(this.options)
    }
  }
}
</script>

<style scoped>
.radio {
  margin: 0 auto;
}
select {
  -webkit-appearance: none; /* google */
  -moz-appearance: none; /* firefox */
  appearance: none; /* IE */
  width: 80px;
  height: 30px;
  font-size: 0.9rem;
  color: white;
  text-align: center;
  background-color: black;
  margin-top: 2.3%;
}
.ctrlBox {
  position: relative;
  color: white;
  width: 60%;
  margin-left: 40%;
  display: flex;
  text-align: center;
  font-size: 1.2rem;
}
input {
  background-color: #21add0;
  color: white;
  border: solid white 1px;
  height: 30px;
  margin-top: 2.3%;
  margin-left: 5%;
  width: 10%;
}
.delBox {
  position: absolute;
  background-color: black;
  z-index: 999999;
  display: flex;
  flex-wrap: wrap;
  width: 30%;
  margin: 0 auto;
}
.delBox p {
  margin: 0 auto;
}
.delBox img {
  margin-left: 30%;
}
.delBox input {
  margin-top: 6%;
  font-size: 1.1rem;
  width: 45%;
}
</style>
