<template>
  <v-app style="background:#66A5AD;">
    <v-app-bar app clipped-left color="#003B46" class="white--text">
      <v-toolbar-title>Budget Tracker</v-toolbar-title>
    </v-app-bar>

    <v-content>
      <v-container>
        <v-card color="#07575B">
          <v-card-title style="background:#003B46">
            <v-col cols="8">
              <span class="white--text">Monthly Budget</span>
            </v-col>
            <v-col cols="4">
              <v-row class="white--text" justify="end">${{budget}}</v-row>
            </v-col>
          </v-card-title>
          <v-divider class="mx-4"></v-divider>
          <v-container>
            <v-row
              align="center"
              justify="center"
              class="white--text header-font"
            >Daily Budget: ${{ dailyBudget }}</v-row>

            <v-row align="center" justify="center">
              <v-text-field
                label="Monthly Budget"
                placeholder="Monthly Budget"
                class="ma-3 pa-6"
                type="number"
                v-model="budget"
                outlined
                dark
              ></v-text-field>
            </v-row>
            <v-row class="ml-3 mr-3">
              <v-col cols="12" sm="6">
                <span class="progress-labels">Daily Spent</span>
                <br />
                <v-progress-linear color="#66A5AD" :height="35" :value="dailyPercent" striped></v-progress-linear>
                <span class="progress-under-labels">${{dailySpent}}/${{dailyBudget}}</span>
                <br />
              </v-col>
              <v-col cols="12" sm="6">
                <span class="progress-labels">Monthly Spent</span>
                <br />
                <v-progress-linear color="#66A5AD" :height="35" :value="monthlyPercent" striped></v-progress-linear>
                <span class="progress-under-labels">${{monthlySpent}}/${{budget}}</span>
                <br />
              </v-col>
            </v-row>
          </v-container>
        </v-card>

        <v-card color="#07575B" class="mt-5">
          <v-card-title style="background:#003B46">
            <v-col cols="8">
              <span class="white--text">Transactions</span>
            </v-col>
            <v-col cols="4">
              <v-row class="white--text" justify="end">
                <v-btn @click="transactionOverlay = true" class="mx-2" fab dark color="#66A5AD">
                  <v-icon dark>mdi-plus</v-icon>
                </v-btn>
              </v-row>
            </v-col>
          </v-card-title>
          <v-divider class="mx-4"></v-divider>
          <v-container>
            <v-row align="center" justify="center" class="white--text header-font">
              <Transactions @update="updateTransactions()" :transactions="transactions" />
            </v-row>
          </v-container>
        </v-card>
        <v-overlay :value="transactionOverlay">
          <v-card color="#07575B">
            <v-card-title style="background:#003B46">
              <v-col cols="9">
                <span class="white--text">Add Transaction</span>
              </v-col>
              <v-col cols="3">
                <v-row class="white--text" justify="end">
                  <v-btn @click="transactionOverlay = false" class="mx-2" fab dark color="#66A5AD">
                    <v-icon dark>mdi-close</v-icon>
                  </v-btn>
                </v-row>
              </v-col>
            </v-card-title>
            <v-text-field
              label="Description"
              placeholder="Description"
              class="ma-3 pa-6"
              v-model="desc"
              outlined
              dark
            ></v-text-field>
            <v-text-field
              label="Amount ($)"
              placeholder="Amount ($)"
              class="ma-3 pa-6 mb-0"
              type="number"
              v-model="amount"
              outlined
              dark
            ></v-text-field>
            <v-row class="white--text" justify="center">
              <v-btn @click="saveTransaction" class="mx-2 mb-3" fab dark color="#66A5AD">
                <v-icon dark>mdi-content-save-move</v-icon>
              </v-btn>
            </v-row>
          </v-card>
        </v-overlay>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import Transactions from "@/components/Transactions.vue";

export default {
  methods: {
    getCurrentDaysInMonth() {
      let now = new Date();
      return new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
    },
    saveTransaction() {
      this.transactions.push({
        date: new Date().toLocaleDateString(),
        desc: this.desc,
        amount: this.transAmount,
        id: "id" + new Date().getTime()
      });
      localStorage.setItem("transactions", JSON.stringify(this.transactions));
    },
    getCurrentMonth() {
      return new Date().getMonth() + 1;
    },
    getCurrentDay() {
      return new Date().getDate();
    },
    updateTransactions() {
      this.transactions = JSON.parse(localStorage.getItem("transactions"));
    }
  },
  data() {
    return {
      daysInCurrentMonth: this.getCurrentDaysInMonth(),
      tmpBudget: null,
      transactionOverlay: false,
      desc: "",
      amount: 0.0,
      transactions: []
    };
  },
  components: {
    Transactions
  },
  computed: {
    dailyBudget() {
      return (this.budget / this.daysInCurrentMonth).toFixed(2);
    },
    transAmount: {
      get() {
        return this.amount;
      },
      set(newAmount) {
        this.amount = newAmount.toFixed(2);
      }
    },
    budget: {
      get() {
        return this.tmpBudget;
      },
      set(newBudget) {
        this.tmpBudget = newBudget;
        localStorage.setItem("budget", this.tmpBudget);
      }
    },
    monthlySpent() {
      let total = 0;
      let currentMonth = String(this.getCurrentMonth());
      if (this.transactions === null) return total;
      this.transactions.forEach(transaction => {
        let splitDate = transaction.date.split("/");
        if (splitDate[0] === currentMonth) {
          total += parseFloat(transaction.amount);
        }
      });
      return total;
    },
    monthlyPercent() {
      return (100 - (this.monthlySpent / this.budget) * 100).toFixed(2);
    },
    dailySpent() {
      let total = 0;
      let currentDate = String(this.getCurrentDay());
      if (this.transactions === null) return total;
      this.transactions.forEach(transaction => {
        let splitDate = transaction.date.split("/");
        if (splitDate[1] === currentDate) {
          total += parseFloat(transaction.amount);
        }
      });
      return total;
    },
    dailyPercent() {
      return (100 - (this.dailySpent / this.dailyBudget) * 100).toFixed(2);
    }
  },
  created() {
    try {
      this.tmpBudget = localStorage.getItem("budget");
    } catch (e) {
      this.tmpBudget = null;
    }
    try {
      this.transactions = JSON.parse(localStorage.getItem("transactions"));
      if (this.transactions === null) {
        localStorage.setItem("transactions", JSON.stringify([]));
        this.transactions = [];
      }
    } catch (e) {
      localStorage.setItem("transactions", JSON.stringify([]));
      this.transactions = [];
    }
  }
};
</script>

<style lang='scss'>
.deep-ocean-inputs {
  background: #66a5ad;
}

.header-font {
  font-size: 2em;
}

.progress-labels {
  color: #ffffff;
}

.progress-under-labels {
  color: #ffffff;
  font-size: 0.75em;
  font-weight: bold;
}
</style>