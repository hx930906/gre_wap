<template>
  <div id="questionDetails" style="height: 100%">
    <view-box ref="viewBox" body-padding-top="46px" body-padding-bottom="51px">
      <x-header slot="header" class="header bg_f">
        <span class="headerTit">{{childData.question.section}}-{{childData.question.source.name}}</span>
        <div slot="overwrite-left" @click="reBack">
          <x-icon style=" fill: #333333;" type="ios-arrow-left" size="30"></x-icon>
        </div>
        <div slot="right">
          <div @click="scQuestion($store.state.userInfo.uid,childData.question.id)" class="scIcon"
               :class="{'scIcon_1':childData.question.collect===0,'scIcon_2':childData.question.collect===1}"></div>
        </div>
      </x-header>
      <!--阅读-->
      <div v-if="childData.question.readArticle" class="readStemWrap">
        <div v-if="typeId===7" class="readStem" :class="{'ellipsis-5':!showAll}">
          <span v-for="(item,index) in childData.question.articles" :class="{'active':curSpan===index}"
                @click="selected($event,index)">{{item}}</span>
        </div>
        <div v-else class="readStem" :class="{'ellipsis-5':!showAll}" v-html="childData.question.readArticle"></div>
        <div class="showAllBtn" @click="showAll=!showAll">{{btnText}}</div>
      </div>
      <!--题干-->
      <div v-if="childData.question.details" class="topicData" v-html="childData.question.details"></div>
      <!--quantity-->
      <div class="quantWrap">
        <div v-if="childData.question.quantityA&&childData.question.quantityA!='<p><br></p>'" style="padding-bottom: 15px">
          <div class="tm"><span class="quantTit">Quantity A</span></div>
          <div class="tm" v-html="childData.question.quantityA"></div>
        </div>
        <div v-if="childData.question.quantityB&&childData.question.quantityB!='<p><br></p>'">
          <div class="tm"><span class="quantTit">Quantity B</span></div>
          <div class="tm" v-html="childData.question.quantityB"></div>
        </div>
      </div>
      <!--选项题型组件-->
      <blank_1 :is="curTemp" v-if="flag" v-bind:testData="childData,typeId,intVal" v-on:getChildAnswer="upAnswer"></blank_1>
      <!--底部-->
      <tabbar slot="bottom" class="vux-1px-t footer">
        <tabbar-item class="vux-1px-r" @on-item-click="toggle(show)">
          <span class="userExit" slot="label">查看解析</span>
        </tabbar-item>
        <!--<tabbar-item>-->
        <!--<span class="userExit" slot="label">下一题</span>-->
        <!--</tabbar-item>-->
      </tabbar>
    </view-box>
    <popup class="window" v-model="show" max-height="60%">
      <div class="answerJs">
        <h1 class="answer">正确答案：{{childData.question.answer}}</h1>
        <div class="answerInfo">
          <div style="margin-bottom: 10px;" v-for="(item,index) in childData.question.analysis" v-html="item.analysisContent"></div>
        </div>
      </div>
    </popup>
    <loading :show="show2" text=""></loading>
    <toast v-model="toastStatu" :text="toastText" width="4rem" type="text" :time="1200" position="bottom"></toast>
  </div>
</template>

<script type="text/ecmascript-6">
  import "../../style/mathquill-0.9.1/mathquill.css";
  import blank_1 from "./child/blank_1"
  import blank_2 from "./child/blank_2"
  import blank_3 from "./child/blank_3"
  import blank_int from "./child/blank_int"
  import {XHeader, Tabbar, TabbarItem, ViewBox, Scroller, Group, Checklist, Popup, Toast, Loading} from 'vux'

  export default {
    name: "questionDetails",
    data() {
      return {
        showAll: false,
        show: false,
        show2: true,
        currentSite: '',
        questionStatus: [],
        totalNum: '',
        childData: {
          question: {
            collect: '',
            optionA: '',
            optionB: '',
            optionC: '',
            optionsA: [],
            optionsB: [],
            optionsC: [],
            quantityA: '',
            quantityB: '',
            readStem: '',
            source: {
              name: ''
            }
          },
          pageType: 1,
          maxVal: null,
          checkAnswer: [],
          trueAnswer: [],
          userAnswer: [],
        },
        flag: false,
        curSpan: '',
        typeId: '',//当前题型
        userAnswer: [],
        useTime: 0,//答题用时
        timeObj: '',//计时器
        curTemp: 'blank_1',//当前组件名
        intVal: {
          trueVal: '',
          userVal: '',
        },
        toastStatu: false,
        toastText: '',
      }
    },
    computed: {
      btnText() {
        if (this.showAll === false) {  //对文字进行处理（枚举）
          return "展开全部"
        } else {
          return "收起"
        }
      }
    },
    components: {
      XHeader, Tabbar, TabbarItem, ViewBox, Scroller, Group, Checklist, Popup, Toast, Loading,
      blank_1, blank_2, blank_3, blank_int
    },
    activated() {
      this.childData.checkAnswer=[];
      this.getData(this.$store.state.userInfo.uid, this.$route.query.qid, this.$route.query.type);
    },
    methods: {
      change(val, label) {
        console.log('change', val, label)
      },
      // 退出
      reBack() {
        this.$router.go(-1);
      },
      getData(uid, qid, type) {
        const _this = this;
        let data = {
          uid: _this.$store.state.userInfo.uid || '',
          questionId: qid,
          type: type,
        };
        _this.axios.get('/cn/wap-api/question-detail', {params: data}).then(function (res) {
          let resType = parseInt(res.data.typeId);
          let trueAnswer = res.data.answer;
          let userAnswer = res.data.useranswer;
          _this.flag = true;
          _this.typeId = resType;
          _this.childData.question = res.data;//子组件数据
          // if (userAnswer) {
          //   if (resType != 2 && resType != 3 && resType != 7 && resType != 10) {
          //     //传入正确答案&&用户答案对比
          //     let userArry = [];
          //     let trueArry = [];
          //     let trueUserArry = [];
          //     userArry.push(...userAnswer);
          //     _this.childData.checkAnswer = getNumber(userArry);
          //     trueArry = getNumber(trueAnswer);
          //     _this.childData.trueAnswer = getTrueArry(trueArry);
          //     trueUserArry = getNumber(userAnswer);
          //     _this.childData.userAnswer = getTrueArry(trueUserArry);
          //     //end
          //   } else {
          //     _this.userAnswer = [];
          //     _this.childData.checkAnswer = [];
          //     _this.childData.userAnswer = [];
          //     _this.childData.trueAnswer = [];
          //   }
          //   // else if (resType === 2 || resType === 3) {
          //   //
          //   // } else if (resType === 7) {
          //   //
          //   // } else if (resType === 10) {
          //   //
          //   // }
          // } else {
          //
          // }
          // 判断typeId单选还是多选赋值给checkList
          if (resType === 1 || resType === 5 || resType === 8) {
            //正确答案为ABCD需要转换为下标值数组传入子组件；
            _this.curTemp = 'blank_1';
            _this.childData.maxVal = 1;
            _this.childData.question.optionsA = resetArry(res.data.optionsA);

          } else if (resType === 2) {
            _this.curTemp = 'blank_2';
            _this.childData.maxVal = 1;
          } else if (resType === 3) {
            _this.curTemp = 'blank_3';
            _this.childData.maxVal = 1;
          } else if (resType === 4) {
            _this.curTemp = 'blank_1';
            _this.childData.maxVal = 2;
            _this.childData.question.optionsA = resetArry(res.data.optionsA);
          } else if (resType === 10) {
            _this.curTemp = 'blank_int';
          } else if (resType === 6 || resType === 9) {
            _this.curTemp = 'blank_1';
            _this.childData.maxVal = 10;
            _this.childData.question.optionsA = resetArry(res.data.optionsA);
          } else {
            _this.childData.maxVal = 1;
          }
          // dom数据渲染完成执行
          _this.$nextTick(() => {
            _this.show2 = false;
            //设置SEO
            document.title = res.data.stem + '_GRE真题机经搜索_gre真题题库_GRE真题题目解析_雷哥GRE题库';
            document.querySelector('meta[name="keywords"]').setAttribute('content',  res.data.stem +'雷哥培训');
            document.querySelector('meta[name="description"]').setAttribute('content',  res.data.stem );
          })

        });
      },
      // 题目收藏
      scQuestion(uid, qid) {
        if (uid) {
          const _this = this;
          let data = {uid: uid, questionId: qid};
          _this.axios.post('/cn/wap-api/user-question-collection', data).then(function (res) {
            if (res.data.code === 1) {
              _this.childData.question.collect = 1;
            }
            if (res.data.code === 2) {
              _this.childData.question.collect = 0;
            }
            _this.$nextTick(function () {
              _this.toastText = res.data.message;
              _this.toastStatu = true;
            })
          });

        } else {
          return false;
        }
      },
      //取子组件值
      upAnswer(data) {
        this.userAnswer = data;
      },
      select(libId, quesId, index) {
        this.getData(libId, quesId, index);
      },
      // 句子点选
      selected(e, index) {
        let dom = e.target;
        this.curSpan = index;
        this.userAnswer = [dom.innerHTML];
      },
      toggle(show) {
        // 判断题型、是否选择答案
        if (this.userAnswer.indexOf('')<0) {
          this.show = show ? false : true;
        } else {
          this.toastText = '请先选择答案';
          this.toastStatu = true;
          return false;

        }

      }
    }
  }

  //正确答案转对应长度数组
  function getTrueArry(val) {
    let rs = [];
    for (let i = 0; i < val.length; i++) {
      rs[val[i]] = val[i];
    }
    return rs;
  }

  //    字母转换为下标
  function getNumber(val) {
    let rs = [];
    for (let i = 0; i < val.length; i++) {
      rs.push(`${val[i].charCodeAt() - 65}`)
    }
    return rs;
  }

  // option 重组
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
  #questionDetails >>> .vux-loading-no-text .weui-toast {
    top: 50%;
    margin-top: -49px; /*no*/
  }

  #questionDetails >>> .weui-toast.vux-toast-bottom {
    bottom: 120px; /*px*/
  }

  #questionDetails >>> .weui-toast_text .weui-toast__content {
    font-size: 14px; /*no*/
    padding: 12px 6px; /*px*/
  }

  #questionDetails >>> .weui-cells {
    font-size: 32px; /*px*/
  }

  #questionDetails >>> .weui-cell__bd {
    font-size: 32px; /*px*/
  }

  #questionDetails >>> .vux-header .vux-header-left {
    top: 10px; /*no*/
  }

  .header {
    width: 100%;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 100;
    border-bottom: 1px solid #a0a0a0; /*px*/
    background: #fff;
  }

  .footer .userExit {
    color: #333333;
    font-size: 32px; /*px*/
  }

  .footer {
    background: #f3f3f3;
    border-top: 1px solid #a0a0a0; /*px*/
  }

  .exitIcon {
    height: 40px; /*px*/
  }

  .scIcon {
    width: 40px; /*px*/
  }

  .headerTit {
    font-size: 32px; /*px*/
    color: #333333;
  }

  .curNum {
    color: #5a5ee4;
  }

  .readStemWrap {
    padding: 26px 20px 30px;
    border-bottom: 16px solid #cccccc;
  }

  .readStem {
    color: #333333;
    box-sizing: border-box;
    font-size: 32px; /*px*/
    line-height: 56px; /*px*/
  }

  .readStem span.active {
    background: #5a5ee4 !important;
    color: #ffffff !important;
  }

  .readStem >>> p, .readStem >>> span {
    color: #333333 !important;
    background: none !important;
    font-size: 32px !important; /*px*/
    line-height: 56px !important; /*px*/
  }

  .showAllBtn {
    padding-top: 18px;
    font-size: 32px; /*px*/
    color: #5a5ee4;
    text-align: center;
  }

  .topicData {
    color: #1b1a1a;
    font-size: 32px; /*px*/
    line-height: 56px; /*px*/
    padding: 26px 20px 30px;
  }

  .topicData >>> p, .topicData >>> span {
    color: #333333 !important;
    background: none !important;
    font-size: 32px !important; /*px*/
    line-height: 56px !important; /*px*/
  }

  .quantWrap {
    padding: 20px 20px;
    font-size: 32px; /*px*/
    color: #333333;
  }

  .quantTit {
    display: inline-block;
    vertical-align: top;
    padding-bottom: 4px;
    border-bottom: 1px solid #333333;
    margin-bottom: 12px;
  }

  .answerJs {
    padding: 0 20px;
    background: #ffffff;
  }

  .window {
    border-top-left-radius: 14px; /*px*/
    border-top-right-radius: 14px; /*px*/
  }

  .answer {
    padding-top: 34px;
    color: #333333;
    font-size: 36px; /*px*/
  }

  .answerInfo {
    padding: 30px 0;
    color: #333333;
    font-size: 30px; /*px*/
    word-break: break-word;
  }

  .testDe {
    display: flex;
    flex-flow: row nowrap;
    justify-content: flex-start;
    padding: 25px 0;
    text-align: center;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
  }

  .testDe li {
    margin: 0 35px; /*px*/
  }

  .testDe li:last-child .linkBtn {
    margin-right: 35px; /*px*/
  }

  .testDe li .linkBtn {
    width: 60px; /*px*/
    height: 60px; /*px*/
    font-size: 34px; /*px*/
    color: #f42e2e;
    line-height: 62px; /*px*/
    border-radius: 6px; /*no*/
    display: block;
  }

  .testDe li .linkBtn.onRight {
    color: #ffffff;
    background: #1a8109;
  }

  .testDe li .linkBtn.onError {
    color: #ffffff;
    background: #f42e2e;
  }

  .testDe li .linkBtn.onNoDo {
    color: #ffffff;
    background: #888888;
  }

  .testDe li .linkBtn.error {
    color: #f42e2e;
    border: 1px solid #f42e2e; /*no*/
  }

  .testDe li .linkBtn.right {
    color: #1a8109;
    border: 1px solid #1a8109; /*no*/

  }

  .testDe li .linkBtn.noDo {
    color: #888888;
    border: 1px solid #888888; /*no*/

  }

  #questionDetails >>> .weui-cells_checkbox .weui-check:checked + .weui-icon-checked:before {
    color: #5a5ee4;
  }

  .scIcon {
    width: 40px; /*px*/
    height: 40px; /*px*/
  }

  .scIcon_1 {
    background: url("/static/images/testDetails/sc_icon.png") no-repeat 0 0;
    background-size: cover;
  }

  .scIcon_2 {
    background: url("/static/images/testDetails/sc_icon2.png") no-repeat 0 0;
    background-size: cover;
  }

</style>
