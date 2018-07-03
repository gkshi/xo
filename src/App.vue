<template>
  <div id="app">

    <bot></bot>

    <header>
      <div class="intro">
        <h1>Крестики-нолики онлайн</h1>
        <a href="https://github.com/gkshi/xo" target="_blank">GitHub</a>
      </div>
    </header>

    <main>
      <div class="intro">

        <div class="mode-toggle">
          <label> <input type="radio" name="mode" value="with_bot" @change="changeGameMode" checked> Игрок против бота
          </label> <label> <input type="radio" name="mode" value="with_user" @change="changeGameMode"> 2 игрока </label>
        </div>

        <section class="game">

          <header>
            <div>{{ activePlayerText }}</div>
            <div>Счет {{ score.x }}:{{ score.o }}</div>
          </header>

          <grid :class="getGridClass()"></grid>

          <footer>
            <a href="#" :class="getButtonClass()" @click.prevent="newRound">Начать заново</a>
          </footer>

        </section>

      </div>
    </main>

  </div>
</template>

<script>
  import Grid from '@/components/Grid'
  import Bot from '@/components/Bot'

  export default {
    name: 'App',

    data: function () {
      return {

        score: {
          x: 0, o: 0
        },

        activePlayer: 'player1', // player1, player2, bot
        activePlayerSign: 'x', // x, o
        activePlayerText: 'Начинайте игру',

        gameMode: 'with_bot', // with_bot, with_user
        gameStatus: 'turn', // turn, draw, win
        gameWinCombination: null, gameStarter: 'x', // x, o

        cells: {
          1: '', 2: '', 3: '', 4: '', 5: '', 6: '', 7: '', 8: '', 9: ''
        },

        winCombinations: [[1, 2, 3], [4, 5, 6], [7, 8, 9], // rows
          [1, 4, 7], [2, 5, 8], [3, 6, 9], // columns
          [1, 5, 9], [3, 5, 7]             // diagonals
        ]

      }
    },

    methods: {

      start: function () {
        if (this.gameMode === 'with_bot' && this.activePlayer !== 'player1') {
          this.$bus.emit('botMove')
        }
      },

      resetSettings: function () {
        // Возвращаем статус игры по умолчанию (turn)
        this.gameStatus = this.$options.data().gameStatus

        this.gameWinCombination = null

        // Очищаем ячейки внутренне
        this.cells = this.$options.data().cells

        // Очищаем ячейки внешне
        this.$bus.emit('clearGrid')
      },

      restart: function (withTurn) {
        // Сбрасываем основные настройки
        this.resetSettings()

        // Если нужно сменить начинающего игрока
        if (withTurn) {
          this.changeActivePlayer()
          this.changePlayerText()
        }
        else {
          this.changeActivePlayer(true)
          this.changePlayerText(true)
        }

        // Размораживаем сетку
        this.$bus.emit('unfreezeGrid')

        // Запускаем новую игру
        this.start()
      },

      newRound: function () {
        // Если игра закончилась (вничью или победой)
        if (this.gameStatus !== 'turn') {
          // Рестарт со сменой начинающего игрока
          this.restart(true)
        }
        else {
          // Обычный рестарт (без смены сторон)
          this.restart()
        }
      },

      makeMove: function (cellOrder) {
        // Заполняем ячейку на поле
        this.cells[cellOrder] = this.activePlayerSign

        // Проверяем выигрышные комбинации
        this.checkGameStatus(cellOrder)

        switch (this.gameStatus) {

          case 'turn':
            // Меняем активного игрока
            this.changeActivePlayer()

            // Меняем текст для игрока
            this.changePlayerText()

            // Выполняем ход бота
            if (this.activePlayer === 'bot') {
              this.$bus.emit('botMove')
            }
            break

          case 'draw':
            // Меняем текст для игрока
            this.activePlayerText = 'Ничья'

            // Замораживаем поле
            this.$bus.emit('freezeGrid')
            break

          case 'win':

            switch (this.gameMode) {
              case 'with_bot':
                (this.activePlayer === 'player1') ? this.activePlayerText = 'Вы победили' : this.activePlayerText =
                  'Победил бот'
                break

              case 'with_user':
                (this.activePlayer === 'player1') ? this.activePlayerText = 'Победил игрок 1' : this.activePlayerText =
                  'Победил игрок 2'
                break

              default:
                break
            }

            // Замораживаем поле
            this.$bus.emit('freezeGrid')
            this.score[this.activePlayerSign]++

            break

          default:
            break
        }

      },

      checkGameStatus: function (lastCellOrder) {
        if (this.isWin(lastCellOrder)) {
          // Победа
          this.gameStatus = 'win'
        }
        else {

          if (this.isGridFull()) {
            // Ничья
            this.gameStatus = 'draw'
          }
          else {
            // Победы нет, игра продолжается
            this.gameStatus = 'turn'
          }

        }
      },

      isWin: function (lastCellOrder) {
        let _this = this, isWin = false, currentSign = this.cells[lastCellOrder], currentCombinations = []

        // Собираем выигрышные комбинации для текущей ячейки
        this.winCombinations.forEach(function (combination) {
          for (let key in combination) {
            if (combination[key] === lastCellOrder) {
              currentCombinations.push(combination)
              break
            }
          }
        })

        // Проверяем каждую комбинацию на выигрыш
        currentCombinations.every(function (combination) {
          let counter = 0

          for (let key in combination) {
            if (_this.cells[combination[key]] === currentSign) {
              counter++
            }
          }

          // Если комбинация выигрышная
          if (counter === 3) {
            // Найдена выигрышная комбинация
            _this.gameWinCombination = combination.join('')
            isWin = true
            return false
          }

          return true
        })

        return isWin
      },

      isGridFull: function () {
        let isFull = true

        for (let key in this.cells) {
          if (!this.cells[key].length) {
            isFull = false
            break
          }
        }

        return isFull
      },

      changeActivePlayer: function (isDefault) {
        // Если нужно сбросить настройку по умолчанию
        if (isDefault) {
          this.activePlayer = this.$options.data().activePlayer
          this.activePlayerSign = this.$options.data().activePlayerSign
          return
        }

        // Если режим "Игрок против бота"
        if (this.gameMode === 'with_bot') {
          (this.activePlayer === 'player1') ? this.activePlayer = 'bot' : this.activePlayer = 'player1'
        }
        else {
          (this.activePlayer === 'player1') ? this.activePlayer = 'player2' : this.activePlayer = 'player1'
        }

        (this.activePlayerSign === 'x') ? this.activePlayerSign = 'o' : this.activePlayerSign = 'x'

      },

      changePlayerText: function () {
        if (this.gameMode === 'with_bot') {
          if (this.activePlayer === 'player1') {
            this.activePlayerText = 'Ваш ход'
          }
          else {
            this.activePlayerText = 'Ход бота'
          }
        }

        if (this.gameMode === 'with_user') {
          if (this.activePlayer === 'player1') {
            this.activePlayerText = 'Ход игрока 1'
          }
          else {
            this.activePlayerText = 'Ход игрока 2'
          }
        }

        //if (this.gameStatus === 'turn') {
        //  switch (this.activePlayerSign) {
        //    case 'x':
        //
        //      break
        //    case 'o':
        //
        //      break
        //  }
        //}

      },

      changeGameMode: function (e) {
        // Сохраняем счет в сесиию
        this.saveScore()

        // Меняем режим игры
        this.gameMode = e.target.value

        this.getScore()
        this.changeActivePlayer(true)
        this.changePlayerText()
        this.restart()
      },

      resetScore: function () {
        // Сбрасываем игровой счет
        this.score = {
          x: 0, o: 0
        }
      },

      getScore: function () {
        let score = null

        switch (this.gameMode) {
          case 'with_bot':
            score = sessionStorage.getItem('score_bot')
            break

          case 'with_user':
            score = sessionStorage.getItem('score_user')
            break

          default:
            break
        }

        score ? this.score = JSON.parse(score) : this.resetScore()
      },

      saveScore: function () {
        let score = JSON.stringify(this.score)

        switch (this.gameMode) {
          case 'with_bot':
            sessionStorage.setItem('score_bot', score)
            break

          case 'with_user':
            sessionStorage.setItem('score_user', score)
            break

          default:
            break
        }
      },

      getGridClass () {
        let className = 'grid'
        this.gameWinCombination ? className += ` grid--win-${this.gameWinCombination}` : ''
        return className
      },

      getButtonClass () {
        let className = 'button'
        this.gameStatus === 'win' ? className += ` button--active` : ''
        return className
      },

      clearSession: function () {
        sessionStorage.removeItem('score_bot')
        sessionStorage.removeItem('score_user')
      }

    },

    computed: {},

    created: function () {
      window.Game = this

      this.clearSession()

      this.$bus.on('strike', (cellOrder) => {
        this.makeMove(cellOrder)
      })

      this.$bus.$on('win', (winner) => {
        this.score[winner]++
      })

      this.start()

    },

    components: {
      Grid, Bot
    }
  }
</script>

<style lang="less">
  * {
    box-sizing: border-box;
  }

  html, body {
    margin: 0;
    padding: 0;
    -moz-osx-font-smoothing: grayscale;
    -webkit-font-smoothing: antialiased;
  }

  body {
    font: normal normal 16px/22px 'Open Sans', sans-serif;
    color: #353535;
    background: url('/assets/bg.png');
  }

  h1 {
    margin: 0;
    text-transform: uppercase;
    font: normal normal 20px/24px 'Open Sans', sans-serif;
  }

  #app {
    display: flex;
    flex-direction: column;
    align-items: stretch;

    height: 100%;

    & > header {
      padding: 20px 50px;

      & > .intro {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
      }
    }

    & > main {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      flex-grow: 1;

      padding: 40px 0 60px;
    }

  }

  .game {
    display: block;
    width: 380px;
    margin: 0 auto;
    background: #2d3e4f;
    color: #fff;
    border-radius: 10px;

    & > header {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      align-items: center;

      padding: 24px 30px;
      border-radius: 10px 10px 0 0;
    }

    & > footer {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
    }

    .button {
      color: #fff;
      text-decoration: none;
      display: block;
      width: 100%;
      height: 100%;
      padding: 24px 30px;
      text-align: center;
      transition: background .15s ease;
      border-radius: 0 0 10px 10px;

      &:hover {
        background: #599ecc;
      }

      &--active {
        background: #599ecc;

        &:hover {
          background: darken(#599ecc, 8%);
        }
      }

    }

  }

  .grid {
    display: grid;
    grid-template: ~'1fr 1fr 1fr / 1fr 1fr 1fr';
    grid-gap: 9px;

    position: relative;

    &::after {
      content: '';
      position: absolute;
      z-index: 0;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
    }

  }

  .mode-toggle {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;

    padding: 0 2px;
    margin: 0 0 20px;
  }

  @media only screen and (max-width: 500px), only screen and (max-device-width: 500px) {

    #app {
      header {
        padding: 20px 20px;
      }

      h1 {
        font: normal normal 14px/18px 'Open Sans', sans-serif;
      }

      .game {
        width: 320px;

        header {
          padding: 24px 20px;
        }

      }
    }

  }
</style>
