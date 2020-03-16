<template>
<!-- 能滚动的盒子 -->
  <div class="viewport" ref="viewport" @scroll="scrollFn">
    <!-- //自己做的一个滚动条 -->
    <div class="scroll-bar" ref="scrollBar"></div>
    <!-- 真实的渲染内容 -->
    <div class="scroll-list" :style="{transform:`translate3d(0,${offset}px,0)`}">
      <div v-for="item in visibleData" :key="item.id" :vid="item.id" ref="items">
        <slot :item="item"></slot>
      </div>
    </div>
  </div>
</template>
<script>
import throttle from 'lodash/throttle'
export default {
  props:{
    size:Number,//当前每一项的高度度
    remain:Number,//可见区域的个数
    items:Array,
    variable:Boolean
  },
  created(){
    this.scrollFn = throttle(this.hancleScroll,200,{leading:false})
  },
  mounted(){
    this.$refs.viewport.style.height = this.size*this.remain+'px'
    this.$refs.scrollBar.style.height  = this.size*this.items.length+'px'
    //如果加载完毕，需要缓存每一项的高度
    this.cacheList()//先记录好，等一会儿滚动到的时候 ，去渲染页面时获取真实dom的高度，未更新缓存内容

    //2、在重新计算滚动太的高度
  },
  data(){
    return{
      start:0,
      end:this.remain,//默认显示8个
      offset:0,
      positions:[]
    }
  },
  computed:{
    //渲染3屏
    //前面预留几个
    preCount(){
      return Math.min(this.start,this.remain)//例如（5，8）//取5
    },
    //后面预留几个
    nextCount(){
      return Math.min(this.remain,this.items.length-this.end)//例如（8，5）取5
    },
    visibleData(){
      let start = this.start - this.preCount
      let end = this.end + this.nextCount
      return this.items.slice(start,end)
    }
  },
  updated(){//页面渲染完成后，需要根据当前展示的数据 更新缓存区的内容
    this.$nextTick(()=>{
      //根据当前显示的 更新缓存中的height bottom ,top 最终更新滚动条的高度
      let nodes = this.$refs.items//获取真实的节点
      if(!(nodes && nodes.length>0)){
        return
      }
      nodes.forEach(node => {
        let {height} = node.getBoundingClientRect()//获取真实的高度
        //我希望改变老的高度
        let id = node.getAttribute('vid')-0//字符串转为number
        let oldHeight = this.positions[id].height
        let val = oldHeight - height //计算当前的高度是否和之前的高度有变化
        if(val){
          this.positions[id].height = height
          this.positions[id].bottom = this.positions[id].bottom-val //bottom增加了
          for(let i= id+1; i<this.positions.length;i++){
            this.positions[i].top = this.positions[i-1].bottom
            this.positions[i].bottom = this.positions[i].bottom-val
          }
        }
      });
      //只要更新过，会算出滚动条的最新高度
      this.$refs.scrollBar.style.height = this.positions[this.positions.length
      -1].bottom+'px'
      //动态的计算滚动条的高度
    })
  },
  methods:{
    cacheList(){
      //缓存每一项的的高度，top 和bottom
      this.positions = this.items.map((item,index) => ({
        height:this.size,
        top:this.size*index,
        bottom:this.size*(index+1)
      }))
    },
    getStartIndex(value){//查找当前滚动条的需要找到的值
      let start = 0
      let end = this.positions.length-1
      let temp = null
      while(start<=end){
        let middleIndex = parseInt((start+end)/2)
        let middleValue = this.positions[middleIndex].bottom//找到当前的中间的那个人的结尾点
        if(middleValue === value){//如果直接找到了，即返回当前的下一个人
          return middleIndex+1
        }else if(middleValue<value){//查找当前的人 ，在右边
          start = middleIndex + 1
        }else if(middleValue>value){//查找当前的人，在左边
          if(temp === null||temp>middleIndex){
            temp = middleIndex//找到范围
          }
          end = middleIndex - 1
        }
      }
      return temp
    },
    hancleScroll(){
      console.log('scroll')
      //1、应该下算出来当前滚过去几个了，当前应该从几个开始显示
      //拿到当前用户滚动的距离
      let scrollTop = this.$refs.viewport.scrollTop
      if(this.variable){//如果有传值variable,说明要使用二分查找找到对应的记录
        this.start = this.getStartIndex(scrollTop)
        this.end = this.start + this.remain
        //位置偏移
        this.offset = this.positions[this.start-this.preCount]?this.positions[this.start-this.preCount].top:0
      }else{
        //2、获取当期应该从第几个开始渲染
        this.start = Math.floor(scrollTop/this.size)// 130/40 = 3.25取3
        this.end = this.start +this.remain
        //定位当前的可视区域，让当前渲染的内容在当前的viewport的可视区域
        //如果有预留渲染，应该把这个位置在向上移动当这么多
        this.offset = this.start *this.size - this.size*this.preCount//让可视区域去调整偏移量
      }
    }
  }
}
</script>
<style lang="scss" scoped>
  .viewport{
    overflow-y: scroll;
    position: relative;
  }
  .scroll-bar{
    position: absolute;
    top:0;
    left:0;
    width:100%;
  }
</style>