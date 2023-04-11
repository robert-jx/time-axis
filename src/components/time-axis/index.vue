<template>
  <section class="time-axis">
    <section class="left" :class="{ 'p-t-30': showTime }">
      <span :class="{ 'red-class': isRedTitle }">{{ title }}</span>
    </section>

    <section class="right">
      <section class="slide-list">
        <!-- 时间刻度 -->
        <div class="slide-ruler" v-if="showTime">
          <div class="scale" v-for="time of timeList" :key="time">
            {{ time }}
          </div>
        </div>
        <div class="slide-box" :class="{ 'm-t-10': !showTime, 'border-line': showBorder }"
          @mousedown.stop="fatherMousedown($event)" :style="isDisabled ? 'background:lightgray;cursor:not-allowed;' : ''">
          <div v-show="domObj.width != 0" class="item"
            :style="`width: ${domObj.width}px; left: ${domObj.left}px; background: ${domObj.bg}`"></div>
          <!-- 创建的时间块数组domArr -->
          <div v-for="(item, key) of domArr" :key="key" class="item"
            :style="`width: ${item.width}px; left: ${item.left}px; background: ${item.bg}`"
            @mousedown.stop="itemMousedown($event, key, item, domArr)" @click="timeBlock(key, item)">
            <i v-show="item.changeTimeShow && iShow === key" class="i-left-class i-class"
              @mousedown.stop="checkI('left', $event, key, item, domArr)"></i>
            <i v-show="item.changeTimeShow && iShow === key" class="i-right-class i-class"
              @mousedown.stop="checkI('right', $event, key, item, domArr)"></i>
            <!-- hover时间展示 -->
            <div style="position: absolute;top: 0;left: 0;width: 100%;height: 100%;"
              v-show="blockShow && !item.changeTimeShow">
              <el-tooltip placement="top">
                <div style="text-align: center" slot="content">
                  {{ item.startH }}:{{ item.startM }} - {{ item.endH }}:{{
                    item.endM
                  }}
                </div>
                <div class="timeHover"></div>
              </el-tooltip>
            </div>

            <!-- 点击时间调整 -->
            <el-popover placement="top" trigger="manual" v-model="item.changeTimeShow" @show="showFun(item)"
              @hide="hideFun(item)">
              <div style="text-align: center">
                <div style="margin: 15px 0">
                  <input size="mini" class="w40" v-model="item.startH" maxlength="2"
                    @input="hhInput('startH', key, item)" />
                  <span style="padding: 0 5px">:</span>
                  <input size="mini" class="w40" v-model="item.startM" maxlength="2"
                    @input="mmInput('startM', key, item)" />
                  <span style="padding: 0 5px">-</span>
                  <input size="mini" class="w40" v-model="item.endH" maxlength="2" @input="hhInput('endH', key, item)" />
                  <span style="padding: 0 5px">:</span>
                  <input size="mini" class="w40" v-model="item.endM" maxlength="2" @input="mmInput('endM', key, item)" />
                </div>
                <div>
                  <el-button-group>
                    <el-button size="mini" type="primary" @click="btns('delete', key, item)">删除</el-button>
                    <el-button size="mini" type="primary" @click="btns('cancel', key, item)">取消</el-button>
                    <el-button size="mini" type="primary" @click="btns('ok', key, item)">确定</el-button>
                  </el-button-group>
                </div>
              </div>
              <div slot="reference" class="timeHover"></div>
            </el-popover>
          </div>
        </div>
      </section>
    </section>
  </section>
</template>

<script>
export default {
  name: 'time-axis',
  props: {
    // 是否展示顶上时间0-24
    showTime: {
      type: Boolean,
      default: true,
    },
    // 左侧展示字段，例：星期日
    title: {
      type: String,
      default: "星期日",
    },
    // 标题是否标红
    isRedTitle: {
      type: Boolean,
      default: false,
    },
    // 展示边框
    showBorder: {
      type: Boolean,
      default: false,
    },
    //控制能做几个时间块
    allNum: {
      type: Number,
      default: 3
    },
    // 默认数据
    defaultArray: {
      type: Array,
      default: () => [],
    }
  },
  data() {
    return {
      // 刻度
      timeList: [
        0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
        20, 21, 22, 23, 24
      ],
      // 总数据
      domArr: [],
      // 总数据数组的下标
      index: 0,
      // 单次操作的对象
      domObj: null,
      // 存根数据
      stubData: {},
      // 是否是选中的时间子块
      loop: 0,
      // 子块的hover是否显示
      blockShow: true,
      // 左右i的显示
      iShow: null,
      // 每个子块的下拉框
      sonVal: null,
      // 小弹框显示前的存根数据，防止修改了时间不点击按钮组直接点空取消弹框而导致时间bug问题
      popoverStubData: [],
      // 下拉框选中的数据
      typeVal: 1,
      // 数据是否正确
      isTrue: true,
      // 是否禁止
      isDisabled: false
    }
  },
  created() {
    this.init();
  },
  methods: {
    // 数据模块-----------------------------------
    init() {
      console.log(1);
      this.domArr = [];
      this.domObj = {
        width: 0, //宽度
        left: 0, //左偏移量
        bg: "#637DEC", //背景色（与背景值关联，对应着下拉框的值
        value: 1, //背景值（与背景色关联，对应着下拉框的值
        startH: 0, //开始时间（时
        startM: 0, //开始时间（分
        endH: 0, //结束时间（时
        endM: 0, //结束时间（分
        changeTimeShow: false, //控制时间显示与时间调整
      };
    },

    // 反馈数据
    submit() {
      this.$emit("onChange", JSON.stringify(this.domArr));
    },

    // 鼠标操作模块-------------------------------------

    // 父节点的按下事件
    fatherMousedown(e) {
      // 控制是否还能创建
      if (this.domArr.length >= this.allNum) {
        return false;
      }
      let op = e.target;
      // 算出鼠标相对元素的位置
      let disX = e.clientX;
      // left
      this.domObj.left = e.layerX;
      // 开始时间
      this.domObj.startH = this.conversionhs("H", this.domObj.left * 2);
      this.domObj.startM = this.conversionhs("M", this.domObj.left * 2);
      //鼠标移动事件
      document.onmousemove = (e) => {
        //用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
        this.domObj.width = e.clientX - disX;
        // 拖动的宽不能大于刻度，否则就等于 刻度-偏移的left = 最终宽
        if (this.domObj.width + this.domObj.left > 720) {
          this.domObj.width = 720 - this.domObj.left;
        }
        // 结束时间
        this.domObj.endH = this.conversionhs(
          "H",
          this.domObj.left * 2 + this.domObj.width * 2
        );
        this.domObj.endM = this.conversionhs(
          "M",
          this.domObj.left * 2 + this.domObj.width * 2
        );
      };
      // 鼠标抬起事件
      document.onmouseup = (e) => {
        if (this.domObj.width > 0) {
          this.domArr.push(this.domObj);
          if (this.domArr.length >= this.allNum) {
            this.isDisabled = true
          }
        }
        this.onSelect(this.typeVal);
        document.onmousemove = null;
        document.onmouseup = null;
        this.submit();
      };
    },

    // 子块节点的长按
    itemMousedown(e, key, item, arr) {
      this.loop = setTimeout(() => {
        this.loop = 0;
        let disX = e.clientX;
        let blockLeft = item.left; //左边偏移量left
        let blockWidth = item.width; //子滑块的宽
        // 控制子块长按时的hover显示
        this.blockShow = false;
        item.changeTimeShow = false;
        let arrCopy = JSON.parse(JSON.stringify(arr));
        // 处理左右滑动的范围
        let leftL = 0; // 往左滑动的范围
        let leftR = 720; // 往右滑动的范围
        let mark = 0;
        for (let i = 0; i < arrCopy.length; i++) {
          if (arrCopy[i].left === item.left) {
            mark = i;
            break;
          }
        }
        if (arrCopy.length === 1) {
          leftL = 0; // 往左滑动的范围
          leftR = 720; // 往右滑动的范围
        } else {
          if (mark === 0) {
            leftL = 0; // 往左滑动的范围
            leftR = arrCopy[mark + 1].left; // 往右滑动的范围
          } else if (mark === arrCopy.length - 1) {
            leftL = arrCopy[mark - 1].left + arrCopy[mark - 1].width;
            leftR = 720;
          } else if (mark !== 0 && mark < arrCopy.length - 1) {
            leftL = arrCopy[mark - 1].left + arrCopy[mark - 1].width;
            leftR = arrCopy[mark + 1].left;
          }
        }
        // 子块鼠标移动事件
        document.onmousemove = (e) => {
          if (blockLeft + (e.clientX - disX) <= leftL) {
            item.left = leftL;
          } else if (item.width + (blockLeft + (e.clientX - disX)) <= leftR) {
            item.left = blockLeft + (e.clientX - disX);
          }
          // 开始时间
          item.startH = this.conversionhs("H", item.left * 2);
          item.startM = this.conversionhs("M", item.left * 2);
          // 结束时间
          item.endH = this.conversionhs("H", item.left * 2 + item.width * 2);
          item.endM = this.conversionhs("M", item.left * 2 + item.width * 2);
        };
        // 子块鼠标抬起事件
        document.onmouseup = (e) => {
          this.blockShow = true;
          this.iShow = null;
          document.onmousemove = null;
          document.onmouseup = null;
        };
        this.submit();
      }, 200);
      return false;
    },
    // 子块节点的单击
    timeBlock(key, item) {
      clearTimeout(this.loop);
      if (this.loop !== 0) {
        this.iShow = key;
        this.sonVal = item.value;
        this.stubData = JSON.parse(JSON.stringify(item));
        // 找到对应的节点，根据条件把对应节点打开或者关闭，但是其它节点要全部关闭
        this.domArr.map((val, j) => {
          if (j == key) {
            item.changeTimeShow = !item.changeTimeShow;
          } else {
            this.domArr[j].changeTimeShow = false;
          }
        });
      }
      this.submit();
      return false;
    },
    // 选中的左右小方块
    checkI(type, e, key, item, arr) {
      this.loop = setTimeout(() => {
        this.loop = 0;
        let disX = e.clientX;
        let blockLeft = item.left; //左边偏移量left
        let blockWidth = item.width; //子滑块的宽
        item.changeTimeShow = false;
        item.value = this.stubData.value;
        let arrCopy = JSON.parse(JSON.stringify(arr));
        // 对需要移动的子块的所在数组重新按照left大小排序
        function sortLeft(a, b) {
          return a.left - b.left;
        }
        arrCopy.sort(sortLeft);
        // 处理左右滑动的范围
        let leftL = 0; // 往左滑动的范围
        let leftR = 720; // 往右滑动的范围
        let mark = 0;
        for (let i = 0; i < arrCopy.length; i++) {
          if (arrCopy[i].left === item.left) {
            mark = i;
            break;
          }
        }
        if (arrCopy.length === 1) {
          leftL = 0; // 往左滑动的范围
          leftR = 720; // 往右滑动的范围
        } else {
          if (mark === 0) {
            leftL = 0; // 往左滑动的范围
            leftR = arrCopy[mark + 1].left; // 往右滑动的范围
          } else if (mark === arrCopy.length - 1) {
            leftL =
              arrCopy[mark - 1].left + arrCopy[mark - 1].width;
            leftR = 720;
          } else if (mark !== 0 && mark < arrCopy.length - 1) {
            leftL =
              arrCopy[mark - 1].left + arrCopy[mark - 1].width;
            leftR = arrCopy[mark + 1].left;
          }
        }
        // 子块鼠标移动事件
        document.onmousemove = (e) => {
          if (type === "left") {
            if (blockLeft + (e.clientX - disX) < leftL) {
              item.left = leftL;
            } else {
              if (blockWidth - (e.clientX - disX) >= 0) {
                item.left = blockLeft + (e.clientX - disX);
                item.width = blockWidth - (e.clientX - disX);
              }
            }
          } else if (type === "right") {
            if (
              blockWidth + (blockLeft + (e.clientX - disX)) <=
              leftR
            ) {
              item.width = blockWidth + (e.clientX - disX);
            }
          }
          // 开始时间
          item.startH = this.conversionhs("H", item.left * 2);
          item.startM = this.conversionhs("M", item.left * 2);
          // 结束时间
          item.endH = this.conversionhs(
            "H",
            item.left * 2 + item.width * 2
          );
          item.endM = this.conversionhs(
            "M",
            item.left * 2 + item.width * 2
          );
        };
        // 左右小方块的鼠标抬起事件
        document.onmouseup = (e) => {
          this.iShow = null;
          if (item.width <= 0) {
            this.domArr.splice(key, 1);
          }
          document.onmousemove = null;
          document.onmouseup = null;
        };
      }, 200);
      return false;
    },

    // 颜色调整
    adjustment(val) {
      if (val == 1) {
        return "#637DEC";
      } else if (val == 2) {
        return "#74B557";
      } else if (val == 3) {
        return "#B83F42";
      } else if (val == 4) {
        return "#E58805";
      } else if (val == 5) {
        return "#B9E2FE";
      } else if (val == 6) {
        return "#AA6FFE";
      }
    },

    // 下拉框选择
    onSelect(code) {
      this.domObj = {
        width: 0, //宽度
        left: 0, //左偏移量
        value: code, //背景值（与背景色关联，对应着下拉框的值
        startH: 0, //开始时间（时
        startM: 0, //开始时间（分
        endH: 0, //结束时间（时
        endM: 0, //结束时间（分
        changeTimeShow: false,
      };
      if (code == 1) {
        this.domObj.bg = "#637DEC";
      } else if (code == 2) {
        this.domObj.bg = "#74B557";
      } else if (code == 3) {
        this.domObj.bg = "#B83F42";
      } else if (code == 4) {
        this.domObj.bg = "#E58805";
      } else if (code == 5) {
        this.domObj.bg = "#B9E2FE";
      } else if (code == 6) {
        this.domObj.bg = "#AA6FFE";
      }
    },

    // 小弹框显示时触发
    showFun(item) {
      this.popoverStubData.push(JSON.parse(JSON.stringify(item)));
    },
    // 小弹框隐藏时触发
    hideFun(item) {
      if (this.iShow != null) {
        item.endH = this.popoverStubData[0].endH;
        item.endM = this.popoverStubData[0].endM;
        item.startH = this.popoverStubData[0].startH;
        item.startM = this.popoverStubData[0].startM;
      }
      this.popoverStubData.splice(0, 1);
    },
    // 剥离自定义指令的小时事件
    hhInput(val, key, item) {
      var reg = RegExp(/\D/);
      if (reg.test(item[val])) {
        item[val] = "00";
      } else if (item[val] >= 24) {
        item[val] = 24;
        if (item[val] == 24) {
          item[val == "startH" ? "startM" : "endM"] = "00";
        }
      }
    },
    // 剥离自定义指令的分钟事件
    mmInput(val, key, item) {
      var reg = RegExp(/\D/);
      // 数字验证
      if (reg.test(item[val]) || item["startH"] == 24 || item["endH"] == 24) {
        item[val] = "00";
      } else if (item[val] > 59) {
        item[val] = 59;
      }
    },
    // 点击时间调整-按钮组
    btns(type, key, item) {
      console.log(type, key, JSON.stringify(item));
      // console.log('==1');
      console.log("domArr", this.domArr);
      if (type == "delete") {
        this.domArr.splice(key, 1);
        this.isDisabled = false
      } else if (type == "cancel") {
        this.domArr[key] = this.stubData;
        item.changeTimeShow = false;
        this.iShow = null;
      } else if (type == "ok") {

        // 判断其他区域是否已存在该时间段
        let obj = item;
        this.isTrue = true;
        this.domArr.forEach((v, index) => {
          if (key != index && ((parseInt(obj.startH) < parseInt(v.endH)) || (parseInt(obj.endH)) > parseInt(v.startH))) {
            this.isTrue = false;
          }
        })
        if (!this.isTrue) {
          this.$message.error('其他区域已有该时间段')
          return
        }
        console.log('obj================>', obj);
        // console.log('点击ok要执行的操作',item,item.startM)
        item.startH = this.timeVerification(item.startH);
        item.startM = this.timeVerification(item.startM);
        item.endH = this.timeVerification(item.endH);
        item.endM = this.timeVerification(item.endM);
        this.operate(key, item);
      }
      this.submit();
    },
    // 按钮组-确定-验证逻辑
    operate(code, item) {
      // 1-时间验证
      if (
        parseInt(item.startH) > parseInt(item.endH) ||
        (parseInt(item.startH) == parseInt(item.endH) &&
          parseInt(item.startM) >= parseInt(item.endM))
      ) {
        this.$message.warning("开始时间必须早于结束时间");
        return;
      }
      // 2-区域变更
      let domArr = this.domArr[code];
      let arrCopy = JSON.parse(JSON.stringify(domArr));
      console.log("code", code);
      console.log("arrCopy", arrCopy);
      console.log('item', item);
      item.left = this.conversionpx("left", item);
      item.width = this.conversionpx("width", item);
      item.changeTimeShow = !item.changeTimeShow;
      item.value = this.sonVal;
      item.bg = this.adjustment(item.value);
      this.iShow = null;
      this.submit();
    },

    // 数据换算

    // 分钟 换算 时,分
    conversionhs(type, timeNum) {
      if (type == "H") {
        return parseInt(timeNum / 60) < 10
          ? "0" + parseInt(timeNum / 60)
          : parseInt(timeNum / 60);
      } else if (type == "M") {
        return parseInt(timeNum % 60) < 10
          ? "0" + parseInt(timeNum % 60)
          : parseInt(timeNum % 60);
      }
    },
    // 时，分 换算成 px
    conversionpx(type, pxNum) {
      if (type == "left") {
        return (
          (parseInt(pxNum.startH) * 60 + parseInt(pxNum.startM)) / 2
        );
      } else if (type == "width") {
        return (
          (parseInt(pxNum.endH) * 60 +
            parseInt(pxNum.endM) -
            (parseInt(pxNum.startH) * 60 +
              parseInt(pxNum.startM))) /
          2
        );
      }
    },

    // 点击确定后验证时间数据是否需要添加 '0' 或者 '00' 或者不变
    timeVerification(time) {
      let timeData = time.toString();
      if (timeData.length == 0) {
        return "00";
      } else if (timeData.length == 1) {
        return `0${timeData}`;
      } else if (timeData.length == 2) {
        return timeData;
      }
    },
  }
}
</script>

<style lang="scss" scoped>
.time-axis {
  width: 800px;
  display: flex;

  // 左侧部分
  .left {
    width: 100px;
    height: 100%;
    line-height: 40px;
    text-align: center;

    .red-class {
      color: #e41823;
    }
  }

  // 右侧部分
  .right {
    width: 720px;
    height: 100%;

    /* 此处总长是720，是一天1440分钟的一半 */
    .slide-list {
      width: 720px;
      color: #303133;

      .slide-ruler {
        position: relative;
        left: -5px;
        width: 730px;
        display: flex;
        justify-content: space-between;
        height: 40px;
        align-items: center;
      }

      .scale {
        text-align: center;
        width: 15px;
        font-size: 12px;
        color: #fff;
      }

      .slide-box {
        width: 100%;
        height: 20px;
        background: #fff;
        display: flex;
        position: relative;
        cursor: pointer;
      }

      .item {
        height: 100%;
        position: absolute;
      }

      .i-class {
        width: 6px;
        height: 20px;
        position: absolute;
        display: inline-block;
        background-position: -5px;
        z-index: 1;
      }

      .i-left-class {
        left: -3px;
      }

      .i-right-class {
        right: -3px;
      }

      .i-class:hover {
        cursor: w-resize;
      }

      .timeHover {
        width: 100%;
        height: 100%;
        display: inline-block;
      }
    }
  }
}

// 全局变量类

.w40 {
  width: 40px !important;
}

.p-10 {
  padding: 10px 0;
}

.p-t-17 {
  padding: 17px 0 0 0;
}

.p-t-30 {
  padding: 30px 0 0 0;
}


.border-line {
  border: 1px solid lightgray;
}

.m-t-10 {
  margin-top: 10px;
}

.has-title {
  height: 70px !important;
}

.no-title {
  height: 40px !important;
}
</style>