<template>
  <el-input
    v-model="currentValue"
    v-bind="{ ...$attrs.column, ...$attrs }"
    v-on="$listeners"
    :disabled="disabled || notEdit"
  >
    <el-button
      style="
        border: 1px solid #66b1ff;
        background-color: #66b1ff;
        color: #ffffff;
      "
      slot="append"
      :disabled="disabled || !notEdit"
      :key="disabled || !notEdit"
      @click="editSecret"
      v-if="showEditBtn"
      >修改
    </el-button>
  </el-input>
</template>

<script>
import { mapGetters } from "vuex";
export default {
  name: "input-secret",
  components: {},
  props: {
    disabled: Boolean,
    editPermission: String,
    value: String,
  },
  data() {
    return {
      notEdit: true,
    };
  },
  //监听属性
  computed: {
    showEditBtn() {
      const { permissions, editPermission, notEdit } = this;
      if (!notEdit) {
        return false;
      }
      if (editPermission) {
        return permissions[editPermission];
      }
      return true;
    },
    currentValue: {
      get: function () {
        return this.value;
      },
      set: function (newValue) {
        this.$emit("input", newValue); // 通过 input 事件更新 model
      },
    },
    ...mapGetters(["permissions"]),
  },
  //监控data中的数据变化
  watch: {},
  created() {},
  mounted() {},
  methods: {
    editSecret() {
      this.notEdit = false;
      this.$emit("input", "");
      this.$emit("secret", true);
    },
  },
};
</script>
<style lang="scss" scoped></style>
