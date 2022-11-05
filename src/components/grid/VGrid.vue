<template>
  <div class="grid-container">
    <div class="grid-view">
      <table>
        <thead>
          <tr>
            <th v-if="selectedCol" style="width: 50px">
              <div class="th-inner">
                <v-checkbox v-model="allSelected"></v-checkbox>
              </div>
            </th>

            <v-th
              ref="th"
              v-for="col in columns"
              :key="col"
              :config="col"
              :allData="allData"
            >
              {{ col.title }}
            </v-th>
          </tr>
        </thead>
        <tbody>
          <v-tr
            v-for="(item, i) in allData"
            :class="selectedIndex[i] ? 'active-tr' : ''"
            :key="item"
            :data="item"
            :columns="columns"
            :selectedCol="selectedCol"
            v-model:selected="selectedIndex[i]"
            @click="handleClick(i)"
            @dblclick="handleDoubleClick(item)"
          >
          </v-tr>
        </tbody>
        <tfoot>
          <v-tfoot :totalCount="50" :allData="allData"></v-tfoot>
        </tfoot>
      </table>
    </div>
    <v-popup-asset
      v-if="isShowPopup"
      :formModel="pram"
      :dataPram="pramData"
    ></v-popup-asset>
  </div>
</template>
<script>
import {
  getCurrentInstance,
  onMounted,
  ref,
  watch,
  reactive,
  nextTick,
  computed,
  onUpdated,
  defineComponent,
} from "vue";
import ColumnType from "@/commons/constant/ColumnType";
import VTh from "./VTh.vue";
import VTr from "./VTr.vue";
import Enum from "@/assets/js/enums/enum.js";
import VPopupAsset from "@/components/popup/VPopupAsset.vue";
import Resource from "@/assets/js/resource/resource.js";
import commonFunction from "@/assets/js/commons/commonFunction.js";
import VCheckbox from "@/components/input/VCheckbox.vue";
import VTfoot from "@/components/grid/VTfoot.vue";

export default defineComponent({
  name: "MsGrid",
  components: { VTh, VTr, VPopupAsset, VCheckbox, VTfoot },
  props: {
    selectedCol: {
      default: false,
      type: Boolean,
    },

    columns: {
      default: [],
    },
    allData: {
      default: [],
    },
    idField: {
      default: "",
    },
    modelValue: {
      default: [],
    },
    selectedField: {
      default: "",
    },
    selectedData: {
      default: [],
    },
    // filters: {
    //   default: [],
    //   type: Array,
    // },
    // active: {
    //   default: null,
    //   type: Object,
    // }
  },

  setup(props, { emit }) {
    const { proxy } = getCurrentInstance();
    const selected = ref([]);
    const isShowPopup = ref(false);
    window.tables = proxy;
    let pram = reactive({
      mode: 0,
      fixed_asset_id: "",
    });
    let pramData = ref({});

    const allSelected = ref(false);
    // Lấy ra những vị trí checked
    const selectedIndex = ref([]);
    const dataPageging = ref([1, 2, 4, 3]);
    const dataSelected = computed(() =>
      selectedIndex.value.map((x, i) => x && proxy.allData[i]).filter((x) => x)
    );

    // Cập nhật dataSelected vào selectedData
    onMounted(() => {
      watch(
        () => proxy.dataSelected,
        (newVal) => {
          emit("update:selected", newVal);
          nextTick(() => {
            emit("update:selectedData", proxy.dataSelected);
          });
        }
      );
    });

    //Theo dõi biến allSelected kiểm tra là true thì lấy tất cả dữ liệu
    watch(
      () => allSelected.value,
      (newVal) => {
        if (newVal) {
          proxy.selectedIndex = proxy.allData.map((x) => true);
        } else {
          proxy.selectedIndex = [];
        }
      }
    );

    /**
     * Xử lý sự kiện double click tr
     *  @author NNNinh(01/11/2022)
     * @pram {object} item dữ liệu asset khi click tr
     */
    const handleDoubleClick = (item) => {
      proxy.pram.mode = Enum.Mode.Update;
      proxy.pramData = item;
      proxy.isShowPopup = true;
    };

    /**
     * Xử lý sự kiện bỏ checked 1 dòng
     *  @author NNNinh(01/11/2022)
     */
    const handleClick = (index) => {
      if (proxy.selectedIndex[index]) {
        proxy.selectedIndex[index] = false;
      } else {
        proxy.selectedIndex[index] = true;
      }
    };

    onMounted(() => {
      proxy.handleSum();
    });

    // format tiền
    function formatMoney(money) {
      money = new Intl.NumberFormat(Resource.LanguageCode.VN, {}).format(money);
      return money;
    }

    // Tính tổng giá trị số lượng nguyên giá,HM/KH lũy kế, giá trị còn lại
    function handleSum(value) {
      let sumA = 0;
      this.allData.forEach((data) => {
        sumA += data[value];
      });
      return formatMoney(sumA);
    }
    return {
      selected,
      handleSum,
      dataSelected,
      handleDoubleClick,
      dataPageging,
      pram,
      isShowPopup,
      allSelected,
      selectedIndex,
      handleClick,
    };
  },
});
</script>
<style lang="scss" scoped>
@import "../../assets/scss/components/v_gid.scss";
.active-tr {
  background-color: rgba(26, 164, 200, 0.2);
}
</style>