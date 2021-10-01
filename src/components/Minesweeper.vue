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
    <div class="row" v-for="(row, r_index) in houses" :key="r_index">
      <div
        class="house"
        :class="{ opened: house.opened }"
        v-for="(house, c_index) in row"
        :key="c_index"
        @click="openHouse(r_index, c_index)"
      >
        <template v-if="house.opened">
          <p v-if="house.hasBomb">💣</p>
          <p v-else-if="house.bombsClose">{{ house.bombsClose }}</p>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
export default {
  setup() {
    let cols = ref(8),
      rows = ref(8);
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
          let bombsClose = 0;
          if (!!houses?.value?.[row_index - 1]?.[house_index - 1]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index + 1]?.[house_index - 1]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index - 1]?.[house_index + 1]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index + 1]?.[house_index + 1]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index]?.[house_index - 1]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index]?.[house_index + 1]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index - 1]?.[house_index]?.hasBomb)
            bombsClose++;
          if (!!houses?.value?.[row_index + 1]?.[house_index]?.hasBomb)
            bombsClose++;
          console.log(row_index, house_index, bombsClose);
          house.bombsClose = bombsClose;
          return house;
        });
      });
    };

    init();
    setUpNumberOfBombsNearby();

    const reset = () => {
      houses.value = [];
      init();
    };

    const openHouse = (row_index, column_index) => {
      houses[row_index][column_index].opened = true;
    };
    return {
      rows,
      cols,
      reset,
      houses,
      openHouse,
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
.opened {
  background-color: lightgrey;
}
</style>