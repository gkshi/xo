<template>
  <div :class="getClass()" v-on="{mouseenter, mouseleave}" @click.prevent="strike"></div>
</template>

<script>
  export default {

    name: 'Cell',

    props: ['order'],

    data () {
      return {
        mark: '',
        empty: true,
        hover: false,
        frozen: false
      }
    },

    watch: {
      mark: function () {
        if (this.length) {
          this.empty = false
        }
      }
    },

    computed: {
      activePlayerSign () {
        return this.$root.$children[0].activePlayerSign
      }
    },

    methods: {

      mouseenter () {
        this.empty && !this.frozen ? this.hover = true : ''
      },

      mouseleave () {
        this.empty && !this.frozen ? this.hover = false : ''
      },

      getClass: function () {
        let className = 'cell'

        if (this.hover) {
          className += ` cell--hover-${this.activePlayerSign}`
        }

        if (this.mark.length) {
          className += ` cell--${this.mark}`
        }

        if (!this.empty || this.frozen) {
          className += ' cell--disabled'
        }

        return className
      },

      strike: function () {
        if (this.empty && !this.frozen) {
          this.empty = false
          this.hover = false
          this.mark = this.$root.$children[0].activePlayerSign
          this.$bus.emit('strike', this.order)
        }
      },

    },

    created: function () {

      this.$bus.$on('clearGrid', () => {
        this.mark = ''
        this.empty = true
        this.frozen = false
        // Ячейка очищена
      })

      this.$bus.$on('freezeGrid', () => {
        this.frozen = true
        // Ячейка заморожена
      })

      this.$bus.$on('unfreezeGrid', () => {
        this.frozen = false
        // Ячейка разморожена
      })

    }

  }
</script>

<style lang="less" scoped>
  @keyframes strike {
    from {
      transform: translate(-50%, -50%) scale(.8);
    }
    50% {
      transform: translate(-50%, -50%) scale(1.2);
    }
    to {
      transform: translate(-50%, -50%) scale(1);
    }
  }

  .cell {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;

    position: relative;
    z-index: 1;
    height: 120px;
    background: #34485d;
    cursor: pointer;

    &::before, &::after {
      content: '';
      position: absolute;
      top: 50%;
      left: 50%;
      transition: opacity .15s ease, visibility .15s ease, transform .15s ease;
      transform: translate(-50%, -50%) scale(1);
      background-position: center;
      background-size: contain;
      background-repeat: no-repeat;
      opacity: 0;
      visibility: hidden;
    }

    &::before {
      width: 54px;
      height: 50px;
      background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48c3ZnIHdpZHRoPSI1NXB4IiBoZWlnaHQ9IjUwcHgiIHZpZXdCb3g9IjAgMCA1NSA1MCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIj4gICAgICAgIDx0aXRsZT5jcm9zczwvdGl0bGU+ICAgIDxkZXNjPkNyZWF0ZWQgd2l0aCBTa2V0Y2guPC9kZXNjPiAgICA8ZGVmcz48L2RlZnM+ICAgIDxnIGlkPSJQYWdlLTEiIHN0cm9rZT0ibm9uZSIgc3Ryb2tlLXdpZHRoPSIxIiBmaWxsPSJub25lIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiPiAgICAgICAgPGcgaWQ9IkFydGJvYXJkIiB0cmFuc2Zvcm09InRyYW5zbGF0ZSgtNTA3LjAwMDAwMCwgLTQ4OC4wMDAwMDApIiBmaWxsPSIjRkZGRkZGIiBmaWxsLXJ1bGU9Im5vbnplcm8iPiAgICAgICAgICAgIDxnIGlkPSJHcm91cCIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoNDY2LjAwMDAwMCwgMjIzLjAwMDAwMCkiPiAgICAgICAgICAgICAgICA8ZyBpZD0iY3Jvc3MiIHRyYW5zZm9ybT0idHJhbnNsYXRlKDQxLjAwMDAwMCwgMjY1LjAwMDAwMCkiPiAgICAgICAgICAgICAgICAgICAgPHBhdGggZD0iTTUzLjc0ODE4MTgsOC4zMDg1NzE0MyBDNTQuMTkwMjU5Nyw3LjQ0Njc1MzI1IDU0LjExNTg0NDIsNi40Njk3NDAyNiA1My43Mjg3MDEzLDUuNjU4NzAxMyBDNTMuODM3NzkyMiw0LjczNTg0NDE2IDUzLjU5NjIzMzgsMy43OTg4MzExNyA1Mi43MjI3MjczLDMuMDgwMTI5ODcgQzUxLjkwOTQ4MDUsMi40MTExNjg4MyA1MS4wOTYzNjM2LDEuNzQxODE4MTggNTAuMjgzNjM2NCwxLjA3MjcyNzI3IEM0OC44NTQ0MTU2LC0wLjEwMzI0Njc1MyA0Ni42MTY0OTM1LC0wLjQ3NDQxNTU4NCA0NS4yMjYzNjM2LDEuMDcyNzI3MjcgQzM4LjkxMjIwNzgsOC4wOTkwOTA5MSAzMi4zMDkzNTA2LDE0LjgyMDI1OTcgMjUuNDY0NDE1NiwyMS4yOTY3NTMyIEMyMC4zNzU0NTQ1LDE2LjU0Mjg1NzEgMTUuMjI0Njc1MywxMS44NTM1MDY1IDEwLjExMDc3OTIsNy4xNTgwNTE5NSBDOC4zMTU5NzQwMyw1LjUwOTYxMDM5IDUuNzE4MDUxOTUsNi4wMjY4ODMxMiA0LjU4ODgzMTE3LDcuOTkwNjQ5MzUgQzIuOTQ0MDI1OTcsOC44MDcwMTI5OSAxLjYxMDc3OTIyLDkuODcxMjk4NyAwLjUwODMxMTY4OCwxMS42ODc3OTIyIEMtMC4wMTY2MjMzNzY2LDEyLjU1MzI0NjggMC4wMjQyODU3MTQzLDEzLjY0NTcxNDMgMC41MDgzMTE2ODgsMTQuNTE0Mjg1NyBDMy44MDk3NDAyNiwyMC40MzM1MDY1IDEwLjE5MTAzOSwyNS42Mzc3OTIyIDE1LjQzNzc5MjIsMzAuNDI3MTQyOSBDMTQuNzM1NzE0MywzMS4wNTU1ODQ0IDE0LjAzODgzMTIsMzEuNjkgMTMuMzIyNTk3NCwzMi4zMDUxOTQ4IEM2LjI1Mzc2NjIzLDM4LjM3NTU4NDQgMS40ODU3MTQyOSw0Mi4yMjcxNDI5IDEwLjMzMTgxODIsNDguNDUxMjk4NyBDMTAuNDUxMTY4OCw0OC41MzUzMjQ3IDEwLjU3Mjg1NzEsNDguNTE2NzUzMiAxMC42OTQwMjYsNDguNTY1ODQ0MiBDMTAuODYzNTA2NSw0OC42NDQwMjYgMTEuMDI2MTAzOSw0OC42OTkwOTA5IDExLjIxNDAyNiw0OC43MyBDMTEuMjcyNDY3NSw0OC43MzIyMDc4IDExLjMyNzAxMyw0OC43NDIwNzc5IDExLjM4NDU0NTUsNDguNzM3NTMyNSBDMTEuNTY2NjIzNCw0OC43NTIzMzc3IDExLjc0NDgwNTIsNDguNzk4NzAxMyAxMS45MjkzNTA2LDQ4Ljc2MjQ2NzUgQzE2LjY4NTMyNDcsNDcuODMgMjEuMTA4MTgxOCw0NC4xMDI5ODcgMjUuMTIxMDM5LDM5Ljk5ODgzMTIgQzI5LjAzMDEyOTksNDMuODUxOTQ4MSAzMy4xOTI1OTc0LDQ3LjUzMjk4NyAzNy43Njk3NDAzLDQ5LjcxNzE0MjkgQzM4Ljg1OTQ4MDUsNTAuMjM3MjcyNyA0MC4xNTg3MDEzLDQ5LjkzNDE1NTggNDAuOTk3NjYyMyw0OS4yMDY2MjM0IEM0Mi40MDQ0MTU2LDQ5LjMxMjg1NzEgNDMuNjg3MDEzLDQ4LjQzODcwMTMgNDQuMzMwNzc5Miw0Ny4xODI0Njc1IEM0NC40MjYxMDM5LDQ3LjEyMzUwNjUgNDQuNDg3NTMyNSw0Ny4wODk4NzAxIDQ0LjU5MDc3OTIsNDcuMDI1MzI0NyBDNDYuNjM1OTc0LDQ1Ljc0NzY2MjMgNDYuODIyODU3MSw0My4yMDMxMTY5IDQ1LjMwOTQ4MDUsNDEuNDQ5MjIwOCBDNDEuODkyMjA3OCwzNy40ODkyMjA4IDM4LjI1OTYxMDQsMzMuNzI0ODA1MiAzNC41NTY0OTM1LDMwLjAyNzE0MjkgQzQxLjIyNDE1NTgsMjMuNTEyMDc3OSA0OS41NDk3NDAzLDE2LjQ4NTU4NDQgNTMuNzQ4MTgxOCw4LjMwODU3MTQzIFoiIGlkPSJTaGFwZSI+PC9wYXRoPiAgICAgICAgICAgICAgICA8L2c+ICAgICAgICAgICAgPC9nPiAgICAgICAgPC9nPiAgICA8L2c+PC9zdmc+);
    }

    &::after {
      width: 46px;
      height: 46px;
      background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48c3ZnIHdpZHRoPSI1NHB4IiBoZWlnaHQ9IjU0cHgiIHZpZXdCb3g9IjAgMCA1NCA1NCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIj4gICAgICAgIDx0aXRsZT5TaGFwZTwvdGl0bGU+ICAgIDxkZXNjPkNyZWF0ZWQgd2l0aCBTa2V0Y2guPC9kZXNjPiAgICA8ZGVmcz48L2RlZnM+ICAgIDxnIGlkPSJQYWdlLTEiIHN0cm9rZT0ibm9uZSIgc3Ryb2tlLXdpZHRoPSIxIiBmaWxsPSJub25lIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiPiAgICAgICAgPGcgaWQ9IkFydGJvYXJkIiB0cmFuc2Zvcm09InRyYW5zbGF0ZSgtNjUwLjAwMDAwMCwgLTQ4Ni4wMDAwMDApIiBmaWxsPSIjRkZGRkZGIiBmaWxsLXJ1bGU9Im5vbnplcm8iPiAgICAgICAgICAgIDxnIGlkPSJHcm91cCIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoNDY2LjAwMDAwMCwgMjIzLjAwMDAwMCkiPiAgICAgICAgICAgICAgICA8ZyBpZD0iY29tcGFzcyIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoMTg4LjAwMDAwMCwgMjY3LjAwMDAwMCkiPiAgICAgICAgICAgICAgICAgICAgPHBhdGggZD0iTS0zLjk5OTY0OTY5LDIzLjI0NDYyOTcgQy0zLjkzMDQ2Mjc1LDguMzUyNDM1ODEgNS42OTg1NDkyOSwtMy41MTAzNTE2IDIxLjI3OTc1MjgsLTMuOTg4MTYxODUgQzM3LjAzMzUzNzYsLTQuNDcxMTI4NTEgNDkuNTY3MDkzLDEwLjI3MjM1MDcgNDkuMTk5NDE1MSwyNi4xOTI3MTk5IEM0OC44NDE1MDM0LDQxLjcwMDU0MDEgMzQuMTE4MDY0Niw1Mi41MzQyOTMyIDE4LjU5NTAzMDksNDkuNDkxMDUzNSBDMTguNTM0MzM4Miw0OS40NzkxMjk2IDE4LjQ3NDcxNjEsNDkuNDY2MTIzOCAxOC40MTYwODA2LDQ5LjQ1MjA4NDQgQzQuODE2MDg1MTUsNDkuMzE0MzM5NyAtNC4wNjYyMjI1NSwzNy41NzQxNTQzIC0zLjk5OTY0OTY5LDIzLjI0NDYyOTcgWiBNNDEuMjAxNTQ2MywyNi4wMDgwNzIyIEM0MS40Njk3Njk4LDE0LjM5NDA1NDMgMzIuMzU4MjI5NiwzLjY3NTk2MTgzIDIxLjUyNDkyOTUsNC4wMDgwODAyOSBDMTAuNjAxMjY3NCw0LjM0MzA2MzI1IDQuMDUwNzU3OTQsMTIuNDEzMTg1MiA0LjAwMDI2Mzk4LDIzLjI4MTc5NjEgQzMuOTUxNDY2NDcsMzMuNzg1MjUyIDEwLjAzMDQ1NjMsNDEuNjQxMTg1IDE5LjAwMzkxNTIsNDEuNDQ5NjAzOSBMMTkuNzYyOTI0MSw0MS40MzMzOTkyIEwyMC40NzUwODkxLDQxLjY5NjQxNjcgQzMxLjE3NTU4ODksNDMuNTg2MTczNyA0MC45NjQ4MjM1LDM2LjI2NDk0NTQgNDEuMjAxNTQ2MywyNi4wMDgwNzIyIFogTTM2LjE1MTkyNjksMjIuMDk2MTUyMSBDMzQuMDk0MTIzNCwxMy4yMzU4NTYzIDI3Ljc1MzI3OCw3LjY4NTkwMTY1IDE5Ljg3MDkwMDUsOC42MDAwNTcwOCBDMTEuNDYyMTc0Miw5LjU3NTI2NTM4IDYuMDQ1NjU3MjEsMTguMzY1NjA4IDguMzAwOTE1OTYsMjYuNzM5MTU3MiBDOS43NzI5Njg2MywzMi4yMDM2OTU3IDEzLjk1NTIwMDksMzUuNzMzNTg0NCAyMC43MTQ1MDI3LDM3LjY4NDI0MDIgQzMwLjk4MTEyOTMsMzguNTY2NzUwNCAzOC4zMDU3MTQyLDMxLjM2ODk2MzEgMzYuMTUxOTI2OSwyMi4wOTYxNTIxIFogTTE4LjkwNTk2ODMsNDUuNDg2NTIyNyBDOS40NTMyMzYwMiw0Mi44NjI2MTMxIDIuODk3MTI3NzYsMzcuNDM1NDQ5IDAuNTc2MjM0ODEsMjguODE5ODU0NCBDLTIuOTA4NzAyNDksMTUuODgwNjMyNCA1LjUxNDA4NzU4LDIuMjExNDc3MzcgMTguOTQ5Mjc2NSwwLjY1MzMyMTMxOCBDMzEuMjc3NTE4NCwtMC43NzY0NDEzNzEgNDEuMDM2NTE4OSw3Ljc2NTMyMzUzIDQzLjk0NDUwNDIsMjAuMjg2MjUxOCBDNDcuNDQyNTIzMywzNS4zNDY0NTI3IDM1LjE2NzIxMyw0Ny4yMDcyMDI0IDE5LjU2ODc3MDEsNDUuNjExNDg5MSBMMTkuMjMyMDgyNCw0NS41NzcwNDYxIEwxOC45MDU5NjgzLDQ1LjQ4NjUyMjcgWiIgaWQ9IlNoYXBlIj48L3BhdGg+ICAgICAgICAgICAgICAgIDwvZz4gICAgICAgICAgICA8L2c+ICAgICAgICA8L2c+ICAgIDwvZz48L3N2Zz4=);
    }

    &--disabled {
      cursor: default;
    }

    &--x {
      &::before {
        opacity: 1;
        visibility: visible;
        animation: strike .2s ease;
      }
    }

    &--o {
      &::after {
        opacity: 1;
        visibility: visible;
        animation: strike .2s ease;
      }
    }

    &--hover-x {
      &::before {
        opacity: .4;
        visibility: visible;
        transform: translate(-50%, -50%) scale(.7);
      }
    }

    &--hover-o {
      &::after {
        opacity: .4;
        visibility: visible;
        transform: translate(-50%, -50%) scale(.7);
      }
    }

  }

  @media only screen and (max-width: 500px), only screen and (max-device-width: 500px) {

    .cell {
      height: 100px;

      &::before {
        width: 42px;
        height: 38px;
      }

      &::after {
        width: 38px;
        height: 38px;
      }

    }

  }
</style>