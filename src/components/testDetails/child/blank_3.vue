<template>
  <div id="blank_3">
    <!--单双空选项-->
    <ul class="blankWrap">
      <li :class="curTab===1?'actived':''" @click="tab(1)">Blank(i)</li>
      <li :class="curTab===2?'actived':''" @click="tab(2)">Blank(ii)</li>
      <li :class="curTab===3?'actived':''" @click="tab(3)">Blank(iii)</li>
    </ul>
    <!--选项-->
    <checklist v-show="curTab===1" class="checkWrap" :pageType="testData.pageType" :trueAnswer="testData.trueAnswer" :userAnswer="testData.userAnswer" :max="testData.maxVal" label-position="right" required :options="testData.question.optionsA"
               @on-change="change"></checklist>
    <checklist v-show="curTab===2" class="checkWrap" :pageType="testData.pageType" :trueAnswer="testData.trueAnswer" :userAnswer="testData.userAnswer" :max="testData.maxVal" label-position="right" required :options="testData.question.optionsB"
               @on-change="change"></checklist>
    <checklist v-show="curTab===3" class="checkWrap" :pageType="testData.pageType" :trueAnswer="testData.trueAnswer" :userAnswer="testData.userAnswer" :max="testData.maxVal" label-position="right" required :options="testData.question.optionsC"
               @on-change="change"></checklist>
  </div>
</template>

<script>
  import {Checklist} from 'vux'

  export default {
    name: "blank_3",
    components: {Checklist},
    data() {
      return {
        answerArry: ['', '',''],
        curTab: 1,
      }
    },
    methods: {
      //选中事件
      change(val, label) {
        let checkText = label.join("");
        if (this.curTab === 1) {
          this.answerArry[0] = checkText;
        }
        if (this.curTab === 2) {
          this.answerArry[1] = checkText;
        }
        if (this.curTab === 3) {
          this.answerArry[2] = checkText;
        }
        this.$emit('getChildAnswer', this.answerArry);
      },
      //选项重构 取答案下标
      test() {
        this.$nextTick(function () {
          let optionsA = this.testData.question.optionsA;
          let optionsB = this.testData.question.optionsB;
          this.resetA = resetArry(optionsA);
          this.resetB = resetArry(optionsB);
          // console.log(this.resetA)
        })

      },
      tab(index) {
        this.curTab = index;
      }
    },
    props: ["testData"]
  }

  function resetArry(val) {
    let rs = [];
    for (let i = 0; i < val.length; i++) {
      rs.push({
        key: `${i}`,
        value: `${val[i]}`,
      })
    }
    return rs
  }
</script>

<style scoped>
  .blankWrap {
    padding: 0 20px;
    display: flex;
    flex-flow: row wrap;
    justify-content: space-around;
    align-items: flex-start;
    padding-bottom: 30px;
  }

  .blankWrap li {
    color: #777777;
    font-size: 32px; /*px*/
    border: 1px solid #777777;
    height: 70px;
    width: 200px;
    line-height: 70px;
    text-align: center;
  }

  .blankWrap li.actived {
    background: #5a5ee4;
    color: #ffffff;
  }
</style>
