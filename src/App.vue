<template lang="pug">
#wrap
  #left
    textarea#editor(
      autofocus="autofocus"
      v-model="text"
    )
    .info 总计:
    .info 大写:
  #right
    #viewer
      p(v-for="item in result") {{ item.length ? item : '&nbsp;' }}
    .info {{total}}
    .info {{numeral}}
</template>

<script>
import Nzh from 'nzh'
import _ from 'lodash'
import math from 'mathjs'

math.config({
  number: 'BigNumber', // Default type of number:
                       // 'number' (default), 'BigNumber', or 'Fraction'
  precision: 64        // Number of significant digits for BigNumbers
})

export default {
  name: 'app',
  data () {
    return {
      text: '',
      result: [],
    }
  },
  methods: {
    mathEval (str) {
      return str ? math.eval(str).toString() : str
    },
    toAccountingNumeral (value) {
      if (!value) {
        return value
      }
      const accountingNumeral = new Nzh({
          ch: '零壹贰叁肆伍陆柒捌玖',
          ch_u: '个拾佰仟万亿',
          other: '负点',
          m_t: '',
          m_z: '整',
          m_u: '元角分',
      })
      return accountingNumeral.toMoney(value)
    },
  },
  computed: {
    total () {
      const result = _.attempt(this.mathEval, _.filter(this.result, item=>{
        return item && item.toString().trim().length > 0
      }).join('+'))
      return _.isError(result) ? '错误' : result
    },
    numeral () {
      return this.toAccountingNumeral(this.total)
    }
  },
  watch: {
    text: {
      handler (val, old) {
        this.result = []
        _.each(val.split('\n'), (item, index) => {
          const result = _.attempt(this.mathEval, item)
          this.result.push(_.isError(result) ? '' : result)
        })
      },
      deep: true,
      immediate: true,
    },
  },
}
</script>

<style>
* {
  padding: 0;
  margin: 0;
  font-family: 'Monaco';
  font-size: 20px;
  /*border: none;
  outline: none;*/
}
#wrap {
  display: flex;
}
textarea {
  /*padding: 5px;*/
  box-sizing: border-box;
}
#viewer {
  height: 100vh;
}
#left {
  height: 100vh;
  flex: 1;
  display: flex;
  flex-direction: column;
}
#right {
  height: 100vh;
  display: flex;
  flex-direction: column;
  text-align: right;
}
#editor {
  flex: 1;
}
#viewer {
  flex: 1;
}
.info {
  text-align: right;
}
</style>
