<template>
  <div id="app">
    <div class="container">
        <MineElevator v-for="(mine, index) in mines" :key="index" 
          v-bind:floors="floors"
          v-bind:mine="mine"
        />
        <div class="buttons">
            <div v-for="floor in reverseMass" :key="floor.numberFloor" class="buttons__item">
                <span class="buttons__item-number">{{ floor.numberFloor }}</span>
                <button @click="addFloorList(floor)"
                  class="buttons__item-button"
                  v-bind:class="{active: floor.buttonActive}">
                  <svg width="15" height="15" viewBox="0 0 15 15" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <circle cx="7.5" cy="7.5" r="3.75"/>
                    <circle cx="7.5" cy="7.5" r="7"/>
                  </svg>
                </button>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import MineElevator from '@/components/MineElevator'
export default {
  name: 'App',
  data() {
    return {
      mines: [],
      floors: [],
      orderVisitFloor: [],
      elevatorsStatusStart: false
    }
  },

  methods: {

    // ДОБАВЛЕНИЕ ЭТАЖА В СПИСОК ПОСЕЩАЕМЫХ ЭТАЖЕЙ
    addFloorList: function(floor) {
      for (let i = 0; i < this.orderVisitFloor.length; i++) {
        if (this.orderVisitFloor[i] === floor.numberFloor) {
          return;
        }
      }
      console.log(this.elevatorOnFloor(floor.numberFloor));
      if (this.elevatorOnFloor(floor.numberFloor)) {
        return;
      }
      this.orderVisitFloor.push(floor.numberFloor);
      this.saveOrderData();
      floor.buttonActive = true;
      console.log(this.mines);
    },

    // ПОИСК СВОБОДНЫХ ЛИФТОВ
    findFreeElevators: function() {
      let freeElevators = [];
      this.mines.forEach(mine => {
        if (mine.freeElevator) {
          freeElevators.push(mine);
        }
      });
      return freeElevators;
    },

    // ПОЛУЧЕНИЕ БЛИЖАЙШЕГО ИЗ СВОБОДНЫХ ЛИФТОВ
    getNearestElevator: function(freeElevators, floor) {
      let nearestElevator = freeElevators[0];
      for (let i = 0; i < freeElevators.length - 1; i++) {
        if (Math.abs(freeElevators[i].currentFloor - floor) > 
          Math.abs(freeElevators[i + 1].currentFloor - floor)) {
          nearestElevator = freeElevators[i + 1];
        }
      }
      return nearestElevator;
    },

    // ПЕРЕМЕЩЕНИЕ ЛИФТА НА НУЖНЫЙ ЭТАЖ
    moveFloor: function(elevator) {
      let timerId = setInterval(() => {
        if (elevator.currentFloor < elevator.onFloor) {
          elevator.currentHeight += 0.25;
        } else if (elevator.currentFloor > elevator.onFloor) {
          elevator.currentHeight -= 0.25;
        } else {
          return false;
        }
        if (elevator.currentHeight === (Math.ceil(100 / this.floors.length * (elevator.onFloor - 1)))) {
          elevator.stoped = true;
          clearInterval(timerId);
          elevator.orderVisitFloor.shift();
          elevator.currentFloor = elevator.onFloor;
          this.saveData();
          setTimeout(() => {
            this.floors[elevator.onFloor-1].buttonActive = false;
            elevator.stoped = false;
            if (elevator.orderVisitFloor.length === 0)
              elevator.freeElevator = true;
            elevator.direction = 'stop';
            this.saveData();
          }, 3000);
        }
      }, 250 / (100 / this.floors.length));
    },

    // ПОЛУЧЕНИЕ ДАННЫХ ИЗ КОНФИГУРАЦИОННОГО ФАЙЛА
    getData: async function() {
      let response = await fetch(`data.json`);
      return await response.json();
    },

    // РАСПРЕДЕЛИТЕЛЬ ПОСЕЩАЕМЫХ ЭТАЖЕЙ ПО ЛИФТАМ
    distributor: function() {
      if (this.orderVisitFloor.length !== 0) {
        if (this.findFreeElevators().length !== 0) {
          const floorNumber = this.orderVisitFloor[0];
          const nearestElevator = this.getNearestElevator(this.findFreeElevators(), floorNumber);
          nearestElevator.orderVisitFloor.push(floorNumber);
          this.orderVisitFloor.shift();
          nearestElevator.freeElevator = false;
          nearestElevator.onFloor = floorNumber;
          if (nearestElevator.currentFloor < floorNumber) {
            nearestElevator.direction = 'move-up';
          } else {
            nearestElevator.direction = 'move-down';
          }
          this.moveFloor(nearestElevator)
        }
      }
    },

    // НЕ ДОБАВЛЯЕМ В СПИСОК ПОСЕЩАЕМЫХ ЭТАЖЕЙ, ЕСЛИ ЛИФТ УЖЕ ЕДЕТ
    // НА НУЖНЫЙ ЭТАЖ ИЛИ НАХОДИТСЯ НА НУЖНОМ ЭТАЖЕ
    elevatorOnFloor: function(floor) {
      for (let i = 0; i < this.mines.length; i++) {
        if (this.mines[i].currentFloor === floor && 
        this.mines[i].onFloor === floor) {
          return true;
        }
      }
      return false;
    },

    // СОХРАНЕНИЕ ДАННЫХ В LocalStorage
    saveData: function() {
      localStorage.setItem('floors', JSON.stringify(this.floors));
      localStorage.setItem('mines', JSON.stringify(this.mines));
    },

    // СОХРАНЕНИЕ ДАННЫХ ПОРЯДКА ПОСЕЩЕНИЯ НЕРАСПРЕДЕЛЁННЫХ ЭТАЖЕЙ
    saveOrderData: function() {
      localStorage.setItem('orderVisitFloor', JSON.stringify(this.orderVisitFloor));
    },

    // ЗАПОЛНЯЕМ МАССИВ ЛИФТОВ ДАННЫМИ
    createMines: function(count, mass=false) {
      if (mass) {
        for (let i = 0; i < mass.length; i++) {
          this.mines.push({
            currentFloor: mass[i].currentFloor,
            currentHeight: mass[i].currentHeight,
            freeElevator: (mass[i].currentFloor === mass[i].onFloor ? true : mass[i].freeElevator),
            direction: (mass[i].currentFloor === mass[i].onFloor ? 'stop' : mass[i].direction),
            onFloor: mass[i].onFloor,
            stoped: (mass[i].currentFloor === mass[i].onFloor ? false : mass[i].stoped),
            orderVisitFloor: mass[i].orderVisitFloor
          });
        }
        for (let i = 0; i < this.mines.length; i++) {
          if (this.mines[i].direction === 'stop') {
            this.floors[this.mines[i].currentFloor - 1].buttonActive = false;
          }
        }
      } else {
        for (let i = 0; i < count; i++) {
          this.mines.push({
            currentFloor: 1,
            currentHeight: 0,
            freeElevator: true,
            direction: 'stop',
            onFloor: 0,
            stoped: false,
            orderVisitFloor: []
          });
        }
      }
    },

    // ЗАПОЛНЯЕМ МАССИВ ЭТАЖЕЙ ДАННЫМИ
    createFloors: function(count, mass=false) {
      if (mass) {
        for (let i = 0; i < mass.length; i++) {
          this.floors.push({
            numberFloor: mass[i].numberFloor,
            buttonActive: mass[i].buttonActive
          });
        }
      } else {
        for (let i = 1; i <= count; i++) {
          this.floors.push({
            numberFloor: i,
            buttonActive: false
          });
        }
      }
    }

  },

  computed: {

    reverseMass() {
      return [...this.floors].reverse();
    },

    // ПРОВЕРКА ЕСТЬ ЛИ СВОБОДНЫЕ ЛИФТЫ
    elevatorFree() {
      for (let i = 0; i < this.mines.length; i++) {
        if (this.mines[i].freeElevator) {
          return true;
        }
      }
      return false;
    },

  },

  watch: {

    orderVisitFloor: function() {
      this.distributor();
    },

    // ЕСЛИ ЕСТЬ СВОБОДНЫЕ ЛИФТЫ, РАСПРЕДЕЛЯЕМ ПО НИМ ЭТАЖИ
    elevatorFree: function(value) {
      if (value) {
        this.distributor();
      }
    }

  },

  beforeMount() {
    this.getData().then((response) => {
      if (localStorage.getItem('floors')) {
        const floors = JSON.parse(localStorage.getItem('floors'));
        if (floors.length === response.countFloor) {
          this.createFloors(floors.length, floors);
          if (localStorage.getItem('orderVisitFloor')) {
            const orderVisitFloor = JSON.parse(localStorage.getItem('orderVisitFloor'));
            if (orderVisitFloor.length !== 0) {
              orderVisitFloor.forEach(item => {
                this.orderVisitFloor.push(item);
              });
            }
          }
        } else {
          localStorage.clear();
        }
      } else {
        this.createFloors(response.countFloor);
      }
      if (localStorage.getItem('mines')) {
        const mines = JSON.parse(localStorage.getItem('mines'));
        if (mines.length === response.countMine) {
          this.createMines(mines.length, mines);
          this.mines.forEach(mine => {
            if (mine.currentFloor !== mine.onFloor) {
              this.moveFloor(mine);
            } else {
              mine.direction = 'stop';
            }
          });
        } else {
          localStorage.clear();
        }
      } else {
        this.createMines(response.countMine);
      }
    }).catch((e) => {
      console.log(e);
      if (localStorage.getItem('floors')) {
        const floors = JSON.parse(localStorage.getItem('floors'));
        if (floors.length === 7) {
          this.createFloors(floors.length, floors);
        }
      } else {
        this.createFloors(7);
      }
      if (localStorage.getItem('mines')) {
        const mines = JSON.parse(localStorage.getItem('mines'));
        if (mines.length === 7) {
          this.createMines(mines.length, mines);
          this.mines.forEach(mine => {
            if (mine.currentFloor !== mine.onFloor) {
              this.moveFloor(mine);
            } else {
              mine.direction = 'stop';
            }
          });
        }
      } else {
        this.createMines(4);
      }
    });
  },

  components: {
    MineElevator
  }
}
</script>

<style>

body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

:root {
    --gray: #919191;
    --blue: #00D4D5;
    --orange: #FB8539;
}

div {
    box-sizing: border-box;
}

.container {
    display: flex;
    height: 100vh;
}

.mine, .buttons {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.mine {
    position: relative;
    width: 150px;
}

.buttons {
    width: 100%;
}

.floor, .buttons__item {
    flex: 1;
}

.buttons__item {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    font-weight: 700;
}

.buttons__item-button {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 10px;
    height: 30px;
    width: 30px;
    border-radius: 3px;
    background: none;
    border: solid 1px var(--blue);
    cursor: pointer;
    transition: 0.2s;
}

.buttons__item-button:active {
  background-color: rgba(251, 133, 57, 0.1);
  border: solid 1px var(--orange);
}

.buttons__item-button:active svg circle:first-child {
  fill: var(--orange)
}

.buttons__item-button:active svg circle:nth-child(2) {
  stroke: var(--orange);
}

.buttons__item-button:hover {
  background-color: rgba(0, 212, 213, 0.1);
}

.buttons__item-button.active:hover {
  background-color: rgba(251, 133, 57, 0.1);
}

.buttons__item-button.active {
  border: solid 1px var(--orange);
}

.buttons__item-button svg circle:first-child {
  fill: var(--blue)
}

.buttons__item-button.active svg circle:first-child {
  fill: var(--orange)
}

.buttons__item-button svg circle:nth-child(2) {
  stroke: var(--blue);
}

.buttons__item-button.active svg circle:nth-child(2) {
  stroke: var(--orange);
}

.floor:first-child, .buttons__item:first-child {
    border-top: 1px solid var(--gray);
}

.floor, .buttons__item {
    border-bottom: 1px solid var(--gray);
}

.floor {
    border-right: 2px solid var(--gray) !important;
    border-left: 2px solid var(--gray) !important;
}

.buttons__item {
    padding: 10px;
}

.elevator {
    position: absolute;
    left: 2px;
    width: calc(100% - 4px);
    background-color: var(--blue);
}

.elevator-pause {
  animation-duration: 3s;
  animation-name: elevator-pause;
}

@keyframes elevator-pause {

  12.5% {
    opacity: 0.2;
  }

  25% {
    opacity: 1;
  }

  37.5% {
    opacity: 0.2;
  }

  50% {
    opacity: 1;
  }

  62.5% {
    opacity: 0.2;
  }

  75% {
    opacity: 1;
  }

  87.5% {
    opacity: 0.2;
  }

}

.elevator__indication {
  display: none;
  justify-content: center;
  align-items: center;
  font-weight: 700;
  color: #fff;
  max-width: 50px;
  margin: 5px auto 0 auto;
  padding: 5px;
  border-radius: 3px;
  background-color: rgba(0, 0, 0, 0.5);
}

.elevator__indication.move-up, .elevator__indication.move-down {
  display: flex;
}

.elevator__indication.move-up .arrow-down,
.elevator__indication.move-down .arrow-up {
  display: none;
}

.elevator__indication-floor {
  font-size: 13px;
  margin-left: 5px;
}

</style>
