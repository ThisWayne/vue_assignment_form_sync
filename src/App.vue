<template>
  <div class="container">
    <h1>Form {{selectedFormInfoId}}</h1>
    <div class="my-1">
      <label for="selectFormId">Jump to</label>
      <select
        id="selectFormId"
        class="mx-1"
        v-bind:value="selectedFormInfoId"
        v-on:change="handleFormIdSelectChange"
      >
        <option
          v-for="formInfo in formInfos"
          v-bind:key="formInfo.id"
          v-bind:value="formInfo.id"
        >{{formInfo.id}}</option>
      </select>
    </div>
    <div>
      <button
        type="button"
        class="btn btn-primary m-1"
        v-on:click="handleLogThisFormClick"
      >Log this form</button>
      <button
        type="button"
        class="btn btn-info m-1"
        v-on:click="handleLogAllFormsClick"
      >Log all forms</button>
      <button
        type="button"
        class="btn btn-danger m-1"
        v-show="formInfos.length > 1"
        v-on:click="handleDeleteThisFormClick"
      >Delete this form</button>
    </div>
    <main class="main my-1">
      <nav>
        <ul class="nav nav-tabs">
          <li class="nav-item">
            <a
              class="nav-link"
              href="personal_info"
              v-bind:class="{active: activeTab === 'personalInfo'}"
              v-on:click.prevent="activeTab = 'personalInfo'"
            >Personal Info</a>
          </li>
          <li class="nav-item">
            <a
              class="nav-link"
              href="orders"
              v-bind:class="{active: activeTab === 'orders'}"
              v-on:click.prevent="activeTab = 'orders'"
            >Orders</a>
          </li>
          <li class="nav-item">
            <a
              class="nav-link"
              href="Overview"
              v-bind:class="{active: activeTab === 'overview'}"
              v-on:click.prevent="activeTab = 'overview'"
            >Overview</a>
          </li>
        </ul>
      </nav>
      <div class="tab-content container">
        <div v-show="activeTab === 'personalInfo'">
          <FormPersonalInfo
            v-bind.sync="currentFormInfo ? currentFormInfo.personal_info: undefined"
            v-on:onNoFixedAddressClick="handleNoFixedAddressClick"
            v-on:onJobSelectChange="handleJobSelectChange"
          ></FormPersonalInfo>
        </div>
        <div v-show="activeTab === 'orders'">
          <FormOrders
            v-bind.sync="currentFormInfo ? currentFormInfo.orders : undefined"
            v-bind:appleCountTick="appleCountTick"
            v-on:onApplePlusMinusClick="handleApplePlusMinusClick"
            v-on:onAppleCountInput="handleAppleCountInput"
            v-on:onBananaCondimentsCheck="handleBananaCondimentsCheck"
          ></FormOrders>
        </div>
        <div v-show="activeTab === 'overview'">
          <FormOverview v-bind.sync="currentFormInfo"></FormOverview>
        </div>
      </div>
    </main>
    <Pagination
      class="my-1"
      v-bind:formInfos="formInfos"
      v-bind:currentFormInfoIndex="currentFormInfoIndex"
      v-on:onPrevPageClick="handlePrevPageClick"
      v-on:onPageClick="handlePageClick"
      v-on:onNextPageClick="handleNextPageClick"
    ></Pagination>
  </div>
</template>

<script>
import testData from "./apiData/testData.json";
import FormPersonalInfo from "./components/FormPersonalInfo.vue";
import FormOrders from "./components/FormOrders.vue";
import FormOverview from "./components/FormOverview.vue";
import Pagination from "./components/Pagination.vue";

const apiData = testData;

export default {
  name: "App",
  components: {
    FormPersonalInfo,
    FormOrders,
    FormOverview,
    Pagination
  },
  data() {
    return {
      activeTab: "personalInfo",
      currentFormInfoIndex: -1,
      formInfos: [],
      appleCountTick: 0
    };
  },
  computed: {
    currentFormInfo() {
      const { formInfos, currentFormInfoIndex } = this.$data;
      if (formInfos.length === 0 || currentFormInfoIndex === -1) {
        return undefined;
      }
      return formInfos[currentFormInfoIndex];
    },
    selectedFormInfoId() {
      return this.currentFormInfo ? this.currentFormInfo.id : undefined;
    }
  },
  mounted() {
    this.$data.formInfos = apiData;
    if (this.$data.formInfos && this.$data.formInfos.length > 0) {
      this.$data.currentFormInfoIndex = 0;
    }
  },
  methods: {
    handleFormIdSelectChange(e) {
      const { value: selectedFormInfoId } = e.target;
      const { formInfos } = this.$data;
      this.$data.currentFormInfoIndex = formInfos.findIndex(
        x => x.id === selectedFormInfoId
      );
    },
    handleLogThisFormClick() {
      const { currentFormInfoIndex, formInfos } = this.$data;
      console.table(formInfos[currentFormInfoIndex]);
    },
    handleLogAllFormsClick() {
      const { formInfos } = this.$data;
      console.table(formInfos);
    },
    handleDeleteThisFormClick() {
      const { currentFormInfoIndex, formInfos } = this.$data;
      formInfos.splice(currentFormInfoIndex, 1);
      this.$data.currentFormInfoIndex = currentFormInfoIndex % formInfos.length;
    },
    handleNoFixedAddressClick(e) {
      this.currentFormInfo.personal_info.address = "";
    },
    handleJobSelectChange(e) {
      this.currentFormInfo.personal_info.job = e.target.value;
    },
    getValidAppleCount(newAppleCount) {
      let validAppleCount = newAppleCount;
      if (validAppleCount > 100) {
        validAppleCount = 100;
      } else if (validAppleCount < 1) {
        validAppleCount = 1;
      }
      return validAppleCount;
    },
    handleApplePlusMinusClick(num) {
      const { apple_count: appleCount } = this.currentFormInfo.orders;
      this.currentFormInfo.orders.apple_count = this.getValidAppleCount(
        Number(appleCount) + num
      );
    },
    handleAppleCountInput(e) {
      const { value: appleCount } = e.target;
      if (Number(appleCount) >= 100) e.preventDefault();
      this.currentFormInfo.orders.apple_count = this.getValidAppleCount(
        Number(appleCount)
      );
      this.$data.appleCountTick++; // force the input to re-render
    },
    handleBananaCondimentsCheck(e) {
      const [{ checked, value }, { banana_condiments }] = [
        e.target,
        this.currentFormInfo.orders
      ];
      if (checked) {
        banana_condiments.push(value);
      } else {
        banana_condiments.splice(banana_condiments.indexOf(value), 1);
      }
    },
    handlePrevPageClick(e) {
      const { currentFormInfoIndex } = this.$data;
      this.$data.currentFormInfoIndex = Math.max(currentFormInfoIndex - 1, 0);
    },
    handlePageClick(index) {
      this.$data.currentFormInfoIndex = index;
    },
    handleNextPageClick(e) {
      const { currentFormInfoIndex, formInfos } = this.$data;
      this.$data.currentFormInfoIndex = Math.min(
        currentFormInfoIndex + 1,
        formInfos.length - 1
      );
    }
  }
};
</script>
<style lang="scss" scoped>
.main {
  border: 1px solid #ccc;
}
</style>
