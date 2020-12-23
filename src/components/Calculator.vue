<template>
  <v-container>
    <v-row class="text-center pb-10">
      <v-col cols="12">
        <h1>Potential Overcharge Calculator</h1>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" md="6" v-for="(tenant, index) in tenants" :key="index">
        <v-card class="pa-6" elevation="4">
          <v-form>
            <v-text-field prepend-inner-icon="mdi-account" label="Tenant Name" v-model="tenant.name"></v-text-field>
            <v-menu
              ref="menu"
              v-model="menu"
              :return-value.sync="tenant.date"
              transition="scale-transition"
              offset-y
              min-width="290px"
            >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="tenant.date"
                    label="Lease Start Date"
                    prepend-inner-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker v-model="tenant.date" no-title scrollable/>
            </v-menu>
            <div class="d-flex">
              <v-text-field v-model="tenant.lrr" prepend-inner-icon="mdi-currency-usd" label="Registered LRR" class="mr-4"></v-text-field>
              <v-text-field v-model="tenant.pr" prepend-inner-icon="mdi-currency-usd" label="Registered PR" class="mr-4"></v-text-field>
              <v-text-field v-model="tenant.longevity" prepend-inner-icon="mdi-timer-sand-full" label="Longevity" class="mr-4" type="number"></v-text-field>
            </div>
            <div class="d-flex">
              <v-radio-group class="mr-10" v-model="tenant.type">
                <p>Type</p>
                <v-radio label="Renewal" value="r"></v-radio>
                <v-radio label="Vacancy" value="v"></v-radio>
                <v-radio label="2nd Year" value="2"></v-radio>
              </v-radio-group>
              <v-radio-group v-model="tenant.year">
                <p>Year</p>
                <v-radio label="1 Year" value="1"></v-radio>
                <v-radio label="2 Year" value="2"></v-radio>
              </v-radio-group>
            </div>
          </v-form>
          <div class="text-right">
            <v-btn small icon @click="tenants.splice(index, 1)">
              <v-icon>mdi-trash-can-outline</v-icon>
            </v-btn>
          </div>
        </v-card>
      </v-col>
      <v-col cols="12" md="6" class="d-flex justify-center align-center">
        <v-btn outlined color="primary" @click="addTenant">
          <v-icon left>mdi-account-plus</v-icon>Add Tenant</v-btn>
      </v-col>
    </v-row>
    <v-row class="mt-8">
      <v-col class="text-center">
        <v-btn large dark>
          <v-icon left>mdi-calculator</v-icon>Calculate</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'HelloWorld',

    data: () => ({
      menu: '',
      tenants: [
        {

        }
      ]
    }),
    methods: {
      addTenant() {
        this.tenants.push({})
      }
    }
  }
</script>
