# elevator

Приложение эмулирующее работу лифтов.
Состоит из шахты лифтов и кнопок вызова лифта на каждом этаже.

Демо: <https://andrey999k.github.io/elevator-test-task-v2/>

Pet-проект с использованием js-фреймворка Vue.

Используемые технологии:
— HTML
— CSS
— JavaScript
— Vue

В разработке был использован компонентный подход.
Приложение содержит единственный компонент **MineElevator** — лифт.
Компонент **MineElevator** принимает 2 пропса:
1) floors — массив с данными этажей
2) mine — объект лифта

Объект массива **floors** содержит данные об номере этажа и о статусе кнопки лифта.
Объект лифта **mine** содержит поля:
1) currentFloor — текущий с которого двигается или на котором находится лифт
2) currentHeight — высота на которой находится лифт
3) direction — направление движения лифта (move-up - вверх, move-down - вниз, stop - стоит на месте)
4) freeElevator — состояние лифта (true - свободен, false - занят)
5) onFloor — этаж, на который движется лифт или текущий этаж, если лифт стоит на месте
6) orderVisitFloor — массив содержащий этажи, которые нужно посетить лифту
7) stoped — состояние высадки/посадки пасажиров (true - производится посадка/высадка)

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
