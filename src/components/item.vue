<template>
  <div class="item">{{item.id}}-{{item.value}}</div>
</template>
<script>
export default {
  
  props:{
    item:Object
  }
}
</script>
<style lang="scss" scoped>
.item{
  border: 1px solid lightblue;
  padding: 20px 0;
}
</style>