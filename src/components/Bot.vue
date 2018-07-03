<template>
  <div hidden>&nbsp;</div>
</template>

<script>
  export default {
    name: 'Bot',

    methods: {

      getOtherSign: function (sign) {
        if (!sign) {
          return
        }

        if (sign === 'x') {
          return 'o'
        }
        else {
          return 'x'
        }
      },

      imitateClick: function (cell, delay) {
        if (!cell) {
          return
        }

        if (!delay) {
          delay = 500
        }

        this.$bus.emit('freezeGrid')

        let _this = this

        setTimeout(function () {
          _this.$bus.emit('unfreezeGrid')
          // клик в ячейку от лица бота
          document.querySelector('.grid').children[cell - 1].click()
        }, delay)
      },

      move: function () {
        // Алгоритм хода бота
        let ready = false
        let cell = null

        // cells
        let cells = this.$root.$children[0].cells
        let bot_cells = []
        let opponent_cells = [], empty_cells = []

        // signs
        let bot_sign = this.$root.$children[0].activePlayerSign
        let opponent_sign = this.getOtherSign(bot_sign)

        // win combinations
        let win_combinations = this.$root.$children[0].winCombinations
        let possible_bot_combinations = []
        let possible_opponent_combinations = []

        // Пробежка по полю, получаем ячейки свои и соперника
        Object.keys(cells).forEach(function (key) {
          if (cells[key] === bot_sign) {
            bot_cells.push(Number(key))
          }
          else if (cells[key] === opponent_sign) {
            opponent_cells.push(Number(key))
          }
          else {
            empty_cells.push(Number(key))
          }
        })

        win_combinations.forEach(function (combination) {
          let isfit = true

          for (let key in combination) {
            if (cells[combination[key]] === opponent_sign) {
              isfit = false
              break
            }
          }

          if (isfit) {
            possible_bot_combinations.push(combination)
          }
        })

        if (bot_cells.length > 0) {

          // 1. Проверяем свою победную комбинаю (две клетки подряд).
          //    Если есть, то завершаем игру

          possible_bot_combinations.every(function (combination) {
            let counter = 0, emptyCell = null

            for (let key in combination) {
              if (cells[combination[key]] === bot_sign) {
                counter++
              }
              if (!cells[combination[key]].length) {
                emptyCell = combination[key]
              }
            }

            if (counter > 1) {
              // Выигрышная комбинация
              cell = emptyCell
              ready = true
              return false
            }

            return true

          })

        }

        if (!ready) {

          // 2. Проверяем победную комбинацию соперника
          //    Если есть, то не занимаем последнюю ячейку, не даем победить

          win_combinations.forEach(function (combination) {
            let isfit = true

            for (let key in combination) {
              if (cells[combination[key]] === bot_sign) {
                isfit = false
                break
              }
            }

            if (isfit) {
              possible_opponent_combinations.push(combination)
            }
          })

          possible_opponent_combinations.every(function (combination) {
            let counter = 0, emptyCell = null

            for (let key in combination) {
              if (cells[combination[key]] === opponent_sign) {
                counter++
              }
              if (!cells[combination[key]].length) {
                emptyCell = combination[key]
              }
            }

            if (counter > 1) {
              // Выигрышная комбинация соперника
              cell = emptyCell
              ready = true
              return false
            }

            return true

          })

        }

        // 3. Строим линию

        if (!ready) {
          // проверяем центр
          if (!cells[5].length) {
            cell = 5
            ready = true
          }
        }

        if (!ready) {

          // Если есть шанс выиграть
          if (possible_bot_combinations.length) {

            // выбираем рандомную комбинацию
            let rand = Math.floor(Math.random() * (possible_bot_combinations.length))
            let comb = possible_bot_combinations[rand]
            let cll = null

            // пробуем 1, 3, 7, 9
            comb.every(function (cell) {
              if (cell % 2 && !cells[cell].length) {
                cll = cell
                return false
              }
              else if (!cells[cell].length) {
                cll = cell
                return false
              }
              return true
            })

            cell = cll
          }
          else {
            // Нет шанса выиграть, играем рандом до ничьи
            let rand = Math.floor(Math.random() * (empty_cells.length))
            cell = empty_cells[rand]
          }

        }

        // Имитируем клик в ячейку
        cell ? this.imitateClick(cell) : ''
      }

    },

    created: function () {
      this.$bus.on('botMove', this.move)
    }

  }
</script>