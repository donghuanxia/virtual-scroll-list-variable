<template>
  <div id="app">
    <!-- 我需要只显示可是区域 -->
    <!-- 1、告诉我的列表每一项多高，因为我要算出一个滚动条来 -->
    <!-- 2、variable 这个高度不一定多高了 -->
    <VirtualScrollList :size=40 :remain=8 :items="items" :variable="true">
      <Item slot-scope="{item}" :item="item"></Item>
    </VirtualScrollList>
  </div>
</template>

<script>
import VirtualScrollList from './components/virtual-scroll-list'
import Item from './components/item'
import Mock from 'mockjs'
let items = []
let total = 100000
for(let i=0;i<total;i++){
  items.push({id:i,value:Mock.Random.sentence(5,50)})
}
export default {
  name: 'App',
  components: {
    VirtualScrollList,
    Item
  },
  data(){
    return{
      items
    }
  }
}
</script>

<style lang="scss">
*{
  margin:0;
  padding: 0;
  box-sizing: border-box;//边框也会算进高度
}
</style>
