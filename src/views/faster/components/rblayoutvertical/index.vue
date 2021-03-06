<template>
  <div style="width:100%;height:100%;">
    <div :class="layoutVertical" @click="unitSelect" @dblclick="unitEdit" draggable="true" @dragstart="unitMove" @dragleave="unitRemove" v-if="mode === 'dev'">
      <div :class="layoutVerticalInner" :id="item.id" :nodeparent="unitid" v-for="item of grid" :style="itemStyle(item)" :key="item.id" @dragleave="unitRemove">
        <container v-for="item of getChild(item.id)" :renId="item.unitID" :key="item.id"> </container>
      </div>
    </div>
    <div :class="layoutVertical" v-if="mode !== 'dev'">
      <div class="container" :id="item.id" :nodeparent="unitid" v-for="item of grid" :key="item.id" :style="itemStyle(item)">
        <container v-for="item of getChild(item.id)" :renId="item.unitID" :key="item.id"> </container>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import container from "../../builder/page/container.vue"
import dev from "../../builder/mixin/unitmixin"
import bus from "../../builder/core/eventbus"
import { Component, Prop, Provide, Vue } from "vue-property-decorator"
import { RenderType, unitDataPath, UnitType } from "../../builder/core/decorators"
import _ from "lodash"

@Component({
  mixins: [dev],
  components: {
    container
  }
})
@UnitType("layout")
@unitDataPath("rbLayoutVertical")
@RenderType("垂直布局")
export default class RbLayoutVertical extends Vue {
  [x: string]: any
  @Prop({ default: "100%" }) private width: string

  @Prop({
    default: "1",
    dataType: "string",
    valueBus: ["2-1", "3-1", "4-4-4", "3-3-3-3", "0-auto", "100px-auto"],
    des: "子grid的比例如果有空值并且是比例按12网格处理",
    alias: "网格比例"
  } as any)
  private gridConfig: string

  @Prop({ default: false, des: "是否隐藏" } as any) private isHide: boolean
  @Prop({ default: "" }) private name: boolean
  @Prop({ default: "" }) private classname: string
  @Prop({ default: "" }) private ids: string
  private containers = ""
  itemStyle(item) {
    const style = {}
    if (item.height) {
      Object.assign(style, { height: item.height })
    } else if (item.flex) {
      Object.assign(style, { flex: item.flex })
    }
    return style
  }
  get grid() {
    const size = this.gridConfig.split("-")
    const isNull = _.indexOf(size, "auto")
    let isPX = false
    const _this = this
    const gridSize: { id: any; height?: string; flex?: string }[] = []
    let tempId: string[] = []
    if (this.ids) {
      tempId = typeof this.ids === "string" ? this.ids.split(",") : this.ids
    }
    function newid() {
      return "container" + _this.randomString(10)
    }
    const lenChange = size.length - tempId.length
    if (lenChange > 0) {
      const more = _.drop(size, tempId.length)
      more.map(() => {
        tempId.push(newid())
      })
    } else if (lenChange < 0) {
      tempId = _.dropRight(tempId, -lenChange)
    }
    size.map(item => {
      isPX = isPX || item.indexOf("px") > -1
    })

    if (isNull === -1 && !isPX) {
      // 不包含auto && 没有px
      let gridNum = 0
      size.map(item => {
        gridNum += parseInt(item)
      })
      size.map((item, index) => {
        gridSize.push({
          id: tempId[index],
          height: item === "auto" ? "auto" : (Number(item) * 100) / gridNum + "%"
        })
      })
    } else if (isPX) {
      size.map((item, index) => {
        if (item === "auto") {
          gridSize.push({
            id: tempId[index],
            flex: "auto"
          })
        } else {
          gridSize.push({
            id: tempId[index],
            height: item === "auto" ? "auto" : item
          })
        }
      })
    } else {
      size.map((item, index) => {
        gridSize.push({
          id: tempId[index],
          flex: item === "auto" ? "auto" : item
        })
      })
    }
    bus.trigger("UnitChangeProps", {
      unit: _this,
      change: {
        key: "ids",
        value: tempId.join(",")
      }
    })
    return gridSize
  }
  get layoutVertical() {
    let calssname = this.calssname ? "rb-layout-vertical " + this.calssname : "rb-layout-vertical "

    if (this.draggable) {
      calssname = calssname + " unit-developing"
    }
    if (this.unitSelected) {
      calssname = calssname + " unitIsSelected "
    }
    return calssname
  }
  get layoutVerticalInner() {
    return "container drag_Mask " + (this.mode === "dev" ? "developing" : "")
  }
}
</script>
<style lang="scss" scoped>
.rb-layout-vertical {
  width: 100%;
  height: 100%;
  &.unit-developing {
    > .developing {
    }
  }
  &.dev {
    box-shadow: 0 1px 6px rgba(0, 0, 0, 0.117647), 0 1px 4px rgba(0, 0, 0, 0.117647);
    & > div {
      border: 1px solid #ccc;
      padding: 10px;
      min-height: 50px;
    }
  }
  display: flex;
  flex-direction: column;
  & > div {
    width: 100%;
    & :last-child {
    }
  }
}
</style>
