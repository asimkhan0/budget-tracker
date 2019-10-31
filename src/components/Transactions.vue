<template>
  <v-data-table
    :headers="headers"
    :items="transactions"
    :items-per-page="5"
    class="elevation-1"
    style="background:#C4DFE6"
  >
    <template v-slot:item.action="{ item }">
      <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
    </template>
  </v-data-table>
</template>

<script>
export default {
  props: ["transactions"],
  data() {
    return {
      headers: [
        {
          text: "Date",
          align: "left",
          sortable: false,
          value: "date"
        },
        { text: "Description", value: "desc" },
        { text: "Amount ($)", value: "amount" },
        { text: "Actions", value: "action", sortable: false }
      ]
    };
  },
  computed: {},
  methods: {
    deleteItem(item) {
      let newTransactions = this.transactions.filter(transaction => {
        if (transaction.id !== item.id) {
          return transaction;
        }
      });
      localStorage.setItem("transactions", JSON.stringify(newTransactions));
      this.$emit("update");
    }
  }
};
</script>

<style>
.deep-ocean-inputs {
  background: #66a5ad;
}
</style>