<template>
  <el-select
    :placeholder="$attrs.label"
    clearable
    :multiple="multiple"
    v-bind="$attrs"
    v-on="$listeners"
    filterable
    :filter-method="filterMethod"
    :no-data-text="noDataText"
    v-loading="loading"
    element-loading-spinner="el-icon-loading"
    @change="handleChange"
  >
    <el-option
      v-for="item in dic"
      :key="item[innerProps.value]"
      :label="item[innerProps.label]"
      :value="item[innerProps.value]"
    >
    </el-option>
  </el-select>
</template>

<script>
import apipool from "@/api/apipool";
import { list_to_obj } from "@/util/util";
export default {
  name: "el-select-plus",
  components: {},
  props: {
    url: {
      type: String,
      default: "",
    },
    props: {
      type: Object,
      default: () => {
        return {
          label: "label",
          value: "value",
        };
      },
    },
    dicLength: {
      type: Number,
      default: 100,
    },
    multiple: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      dic: [],
      dicAll: [],
      noDataText: "暂无数据",
      loading: false,
      valueToItem: {},
    };
  },
  //监听属性
  computed: {
    innerProps() {
      const { label = "label", value = "value" } = this.props;
      return {
        label,
        value,
      };
    },
  },
  //监控data中的数据变化
  watch: {},
  created() {
    this.getDic();
  },
  mounted() {},
  methods: {
    setDic(list, listMin) {
      const { label: labelKey = "label", value: valueKey = "value" } =
        this.props;
      if (this.multiple) {
        // 多选
        const { value = [] } = this.$attrs;
        if (value.length > 0) {
          const filterList = list
            .filter((item) => {
              return value.includes(item[key]);
            })
            .filter((item) => {
              return !listMin.includes(item);
            });
          this.dic = [...filterList, ...listMin];
          // todo 此处没去重
        } else {
          this.dic = listMin;
        }
      } else {
        // 单选
        const { value = "" } = this.$attrs;
        if (value) {
          if (
            listMin.some((e) => {
              return e[valueKey] === value;
            })
          ) {
            this.dic = listMin;
          } else {
            const findEle = list.find((ele) => {
              return e[valueKey] === value;
            });
            this.dic = findEle ? [findEle, ...listMin] : listMin;
          }
        } else {
          this.dic = listMin;
        }
      }
    },
    handleChange(val) {
      if (Array.isArray(val)) {
        const items = val.map((v) => {
          return this.valueToItem[v] || {};
        });
        this.$emit("change-plus", val, items);
      } else {
        this.$emit("change-plus", val, this.valueToItem[val] || {});
      }
    },
    filterMethod(val) {
      const { label: labelKey = "label", value: valueKey = "value" } =
        this.props;
      this.dic = this.dicAll
        .filter((ele) => {
          return (ele[labelKey] || "").includes(val);
        })
        .slice(0, this.dicLength);
    },
    getDic(params = {}) {
      if (!this.url) {
        return;
      }
      this.loading = true;
      this.noDataText = "加载中...";
      apipool
        .getUrl({
          url: this.url,
          params: {
            ...this.params,
            ...params,
          },
        })
        .then((res) => {
          const list = res.data.data;
          this.dicAll = [...res.data.data];

          const listMin = list.slice(0, this.dicLength);

          this.setDic(list, listMin);

          const { label: labelKey = "label", value: valueKey = "value" } =
            this.props;
          this.valueToItem = list_to_obj(this.dicAll, valueKey);
        })
        .finally(() => {
          this.noDataText = "暂无数据";
          this.loading = false;
        });
    },
  },
};
</script>
<style lang="scss" scoped></style>
