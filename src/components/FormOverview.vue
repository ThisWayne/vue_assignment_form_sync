<template>
  <div class="my-1">
    <span>followings are {{fullname}} order:</span>
    <ul>
      <li>{{orders.apple_count}} apples</li>
      <li v-show="orders.banana_condiments.length > 0">Banana condiments
        <ul>
          <li v-for="condiment in orders.banana_condiments" v-bind:key="condiment">{{condiment}}</li>
        </ul>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  props: {
    personal_info: {
      type: Object,
      default() {
        return {};
      }
    },
    orders: {
      type: Object,
      default() {
        return {
          banana_condiments: []
        };
      }
    }
  },
  computed: {
    fullname() {
      let fullname = "unknown";
      if (!this.$props.personal_info) {
        return fullname;
      }

      const { first_name, last_name } = this.$props.personal_info;
      if (first_name) {
        fullname = last_name ? `${first_name} ${last_name}` : first_name;
      } else if (last_name) {
        fullname = last_name;
      }

      return fullname;
    }
  }
};
</script>

