<template>
  <div>
    <cell v-for="(cell, index) in 9" :order="index + 1" :key="cell.id"></cell>
  </div>
</template>

<script>
  import Cell from '@/components/Cell'

  export default {
    name: 'Grid',

    components: {
      Cell
    }

  }
</script>

<style lang="less" scoped>
  *[class^='grid grid--win-'] {
    &::before {
      content: '';
      position: absolute;
      z-index: 3;
      width: 100%;
      height: 5px;
      background: #8bd473;
      border-radius: 2px;
    }
  }

  .grid {
    &--win-123 {
      &::before {
        width: 80%;
        top: 16%;
        left: 10%;
      }
    }
    &--win-456 {
      &::before {
        width: 80%;
        top: 50%;
        left: 10%;
      }
    }
    &--win-789 {
      &::before {
        width: 80%;
        top: 83%;
        left: 10%;
      }
    }
    &--win-147 {
      &::before {
        width: 80%;
        top: 50%;
        left: -24%;
        transform: rotate(90deg);
      }
    }
    &--win-258 {
      &::before {
        width: 80%;
        top: 50%;
        left: 10%;
        transform: rotate(90deg);
      }
    }
    &--win-369 {
      &::before {
        width: 80%;
        top: 50%;
        left: 44%;
        transform: rotate(90deg);
      }
    }
    &--win-159 {
      &::before {
        top: 50%;
        left: 0;
        transform: rotate(45deg);
      }
    }
    &--win-357 {

      /*.cell {*/
      /*&:nth-child(1), &:nth-child(2),*/
      /*&:nth-child(4), &:nth-child(6),*/
      /*&:nth-child(8), &:nth-child(9) {*/
      /*opacity: .5;*/
      /*}*/
      /*}*/

      &::before {
        top: 50%;
        left: 0;
        transform: rotate(-45deg);
      }
    }
  }
</style>