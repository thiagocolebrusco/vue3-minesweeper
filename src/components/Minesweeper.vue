<template>
  <div>
    <div>
      <label>Rows</label>
      <input type="text" v-model="rows" />
    </div>
    <div>
      <label>Cols</label>
      <input type="text" v-model="cols" />
    </div>
    <div>
      <button @click="reset">Reset</button>
    </div>
    <div>Timer: {{ formattedElapsedTime }}</div>
    <div class="row">
      <div style="width: 50%; float: left;">
        <div class="row" v-for="(row, r_index) in houses" :key="r_index">
          <div
            class="house"
            :class="{ opened: house.opened, exploded: house.exploded }"
            v-for="(house, c_index) in row"
            :key="c_index"
            @click="openHouse(r_index, c_index)"
          >
            <template v-if="house.opened">
              <p v-if="house.hasBomb">💣</p>
              <p v-else-if="house.bombsNearby" :class="'bombsNearby bombsNearby'+house.bombsNearby">{{ house.bombsNearby }}</p>
            </template>
          </div>
        </div>
      </div>
      <div style="width: 50%; float: left;">
        <div class="row" v-for="(row, r_index) in houses" :key="r_index">
          <div
            class="house"
            :class="{ opened: true, exploded: house.exploded }"
            v-for="(house, c_index) in row"
            :key="c_index"
            @click="openHouse(r_index, c_index)"
          >
            <p v-if="house.hasBomb">💣</p>
            <p v-else-if="house.bombsNearby" :class="'bombsNearby bombsNearby'+house.bombsNearby">{{ house.bombsNearby }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue";
const nearbyOptions = [
  [-1, -1],
  [-1, 1],
  [1, -1],
  [1, 1],
  [0, 1],
  [0, -1],
  [1, 0],
  [-1, 0],
];

export default {
  setup() {
    let timer;
    let cols = ref(8),
      rows = ref(8),
      elapsedTime = ref(0);
    let houses = ref([]);

    const init = () => {
      for (let i = 0; i < rows.value; i++) {
        houses.value[i] = [];
        for (let j = 0; j < cols.value; j++) {
          houses.value[i][j] = {
            opened: false,
            hasBomb: Math.random() * (9 - 0) + 0 < 1,
          };
        }
      }
    };

    const setUpNumberOfBombsNearby = () => {
      houses = houses.value.map((row, row_index) => {
        return row.map((house, house_index) => {
          let bombsNearby = 0;

          nearbyOptions.map(([x, y]) => {
            if (!!houses?.value?.[row_index + x]?.[house_index + y]?.hasBomb)
              bombsNearby++;
          });

          house.bombsNearby = bombsNearby;
          return house;
        });
      });
    };

    init();
    setUpNumberOfBombsNearby();

    const formattedElapsedTime = computed(() => {
      const date = new Date(null);
      date.setSeconds(elapsedTime.value / 1000);
      const utc = date.toUTCString();
      return utc.substr(utc.indexOf(":") - 2, 8);
    });
    const startTimer = () => {
      timer = setInterval(() => {
        elapsedTime.value += 1000;
      }, 1000);
    };

    const reset = () => {
      elapsedTime.value = 0;
      houses.value = [];
      init();
      clearInterval(timer);
      startTimer();
    };

    startTimer();

    const openAllHouses = () => {
      houses.forEach((row, row_index) => {
        row.forEach((column, column_index) => {
          column.opened = true;
        })
      })
    }

    const openSpacesNearby = (row_index, column_index) => {
      const tests = [
        { row: row_index - 1, column: column_index - 1 },
        { row: row_index - 1, column: column_index },
        { row: row_index - 1, column: column_index + 1 },
        { row: row_index + 1, column: column_index - 1 },
        { row: row_index + 1, column: column_index },
        { row: row_index + 1, column: column_index + 1 },
        { row: row_index, column: column_index - 1 },
        { row: row_index, column: column_index + 1 },
      ].filter((item) => houses[item.row]?.[item.column]);

      tests.forEach((item) => {
        if(houses[item.row]?.[item.column]?.opened) return;

        houses[item.row][item.column].opened = true;
        if(!houses[item.row]?.[item.column]?.bombsNearby) {
          openSpacesNearby(item.row, item.column);
        }
      })
    }

    const openHouse = (row_index, column_index) => {
      houses[row_index][column_index].opened = true;
      if(houses[row_index][column_index].hasBomb) {
        openAllHouses();
        houses[row_index][column_index].exploded = true;
      } else if(!houses[row_index][column_index].bombsNearby) {
        openSpacesNearby(row_index, column_index);
      }
    };

    return {
      rows,
      cols,
      reset,
      houses,
      openHouse,
      formattedElapsedTime,
    };
  },
};
</script>

<style scoped>
.row {
  clear: both;
}
.house {
  border: 1px black solid;
  height: 50px;
  width: 50px;
  background-color: grey;
  float: left;
}
.house.exploded {
  background-color: red;
}
.opened {
  background-color: lightgrey;
}
.bombsNearby {
  color: orange;
}
.bombsNearby1 {
  color: blue;
}
.bombsNearby2 {
  color: green;
}
.bombsNearby3 {
  color: red;
}
.bombsNearby4 {
  color: purple;
}
</style>
