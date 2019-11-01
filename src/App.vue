<template>
  <div id="app">

    <bot></bot>

    <header>
      <div class="intro">
        <h1>Крестики-нолики онлайн</h1>
        <div class="links">
          <a href="https://github.com/gkshi/xo" target="_blank">GitHub</a>
          <a href="https://gkshi.github.io/" target="_blank">More projects</a>
        </div>
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
        gameWinCombination: null,
        gameStarter: 'x', // x, o

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
        this.gameStatus !== 'turn' ? className += ` button--active` : ''
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
    background: url('data:image/png;base64,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');
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

  .links {
    & > * {
      &:not(:last-child) {
        margin-right: 10px;
      }
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

  footer {
    text-align: center;
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
