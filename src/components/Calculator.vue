<template>
  <div>
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
              <v-text-field
                prepend-inner-icon="mdi-account"
                label="Tenant Name"
                v-model="tenant.name"
              ></v-text-field>
              <v-menu
                ref="menu"
                v-model="menu"
                :close-on-content-click="false"
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
                <v-date-picker v-model="tenant.date" no-title scrollable />
              </v-menu>
              <div class="d-flex">
                <v-text-field
                  v-model="tenant.lrr"
                  prepend-inner-icon="mdi-currency-usd"
                  label="Registered LRR"
                  class="mr-4"
                ></v-text-field>
                <v-text-field
                  v-model="tenant.pr"
                  prepend-inner-icon="mdi-currency-usd"
                  label="Registered PR"
                  class="mr-4"
                ></v-text-field>
                <v-text-field
                  v-model="tenant.longevity"
                  prepend-inner-icon="mdi-timer-sand-full"
                  label="Longevity"
                  class="mr-4"
                  type="number"
                ></v-text-field>
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
            <v-icon left>mdi-account-plus</v-icon>Add Tenant</v-btn
          >
        </v-col>
      </v-row>
      <v-row class="mt-8">
        <v-col class="text-center">
          <v-btn large dark>
            <v-icon left>mdi-calculator</v-icon>Calculate</v-btn
          >
        </v-col>
      </v-row>
    </v-container>
    <v-container fluid>
      <v-row class="mt-8">
        <v-col>
          <p>Calculate</p>
          <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">Tenant</th>
                  <th class="text-left">Lease Start Date</th>
                  <th class="text-left">Registered LRR</th>
                  <th class="text-left">Registered PR</th>
                  <th class="text-left">Longevity</th>
                  <th class="text-left">(r)enewal/(v)acancy/(2)nd Year</th>
                  <th class="text-left">(1) Yr or (2) Yr</th>
                  <th class="text-left">Renewal Row</th>
                  <th class="text-left">Renewal Increase</th>
                  <th class="text-left">Vacancy Row</th>
                  <th class="text-left">Vacancy Increase</th>
                  <th class="text-left">Logevity Increase</th>
                  <th class="text-left">Legal Increase (%)</th>
                  <th class="text-left">Legal Increase ($)</th>
                  <th class="text-left">New Legal Rent</th>
                  <th class="text-left">Potential Overcharge</th>
                  <th class="text-left">
                    Required IAI (Building under 35 Units)
                  </th>
                  <th class="text-left">Required IAI (Building > 35 Units)</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(tenant, index) in tenants" :key="index">
                  <td>{{ tenant.name }}</td>
                  <td>{{ tenant.date }}</td>
                  <td>{{ tenant.lrr }}</td>
                  <td>{{ tenant.pr }}</td>
                  <td>{{ tenant.longevity }}</td>
                  <td>{{ tenant.type }}</td>
                  <td>{{ tenant.year }}</td>
                  <td>{{ calculateRenewal(index) }}</td>
                  <td>{{ renewalIncrease(index) }}</td>
                  <td>{{ vacancyRow(index) }}</td>
                  <td>{{ vacancyIncrease(index) }}</td>
                  <td>{{ longitivityIncrease(index) }}</td>
                  <td>{{ legalIncreasePercent(index) }}</td>
                  <td>{{ legalIncreaseDolar(index) }}</td>
                  <td>{{ newLegalRent(index) }}</td>
                  <td>{{ potentialOverCharge(index) }}</td>
                  <td>{{ requiredIaiLT35(index) }}</td>
                  <td>{{ requiredIaiGT35(index) }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import moment from 'moment'

export default {
  name: 'HelloWorld',

  data: () => ({
    menu: '',
    tenants: [],
    vacancyIncreases: [
      {
        Index: 1,
        startDate: '10/1/01',
        endDate: '9/30/02',
        oneYear: '18.00%',
        twoYear: '20.00%'
      },
      {
        Index: 2,
        startDate: '10/1/02',
        endDate: '9/30/03',
        oneYear: '18.00%',
        twoYear: '20.00%'
      },
      {
        Index: 3,
        startDate: '10/1/03',
        endDate: '9/30/04',
        oneYear: '17.00%',
        twoYear: '20.00%'
      },
      {
        Index: 4,
        startDate: '10/1/04',
        endDate: '9/30/05',
        oneYear: '17.00%',
        twoYear: '20.00%'
      },
      {
        Index: 5,
        startDate: '10/1/05',
        endDate: '9/30/06',
        oneYear: '17.25%',
        twoYear: '20.00%'
      },
      {
        Index: 6,
        startDate: '10/1/06',
        endDate: '9/30/07',
        oneYear: '17.00%',
        twoYear: '20.00%'
      },
      {
        Index: 7,
        startDate: '10/1/07',
        endDate: '9/30/08',
        oneYear: '17.25%',
        twoYear: '20.00%'
      },
      {
        Index: 8,
        startDate: '10/1/08',
        endDate: '9/30/09',
        oneYear: '16.00%',
        twoYear: '20.00%'
      },
      {
        Index: 9,
        startDate: '10/1/09',
        endDate: '9/30/10',
        oneYear: '17.00%',
        twoYear: '20.00%'
      },
      {
        Index: 10,
        startDate: '10/1/10',
        endDate: '9/30/11',
        oneYear: '17.75%',
        twoYear: '20.00%'
      },
      {
        Index: 11,
        startDate: '10/1/11',
        endDate: '9/30/12',
        oneYear: '16.50%',
        twoYear: '20.00%'
      },
      {
        Index: 12,
        startDate: '10/1/12',
        endDate: '9/30/13',
        oneYear: '18.00%',
        twoYear: '20.00%'
      },
      {
        Index: 13,
        startDate: '10/1/13',
        endDate: '9/30/14',
        oneYear: '16.25%',
        twoYear: '20.00%'
      },
      {
        Index: 14,
        startDate: '10/1/14',
        endDate: '6/14/15',
        oneYear: '18.25%',
        twoYear: '20.00%'
      },
      {
        Index: 15,
        startDate: '6/15/15',
        endDate: '9/30/15',
        oneYear: '18.25%',
        twoYear: '20.00%'
      },
      {
        Index: 16,
        startDate: '10/1/15',
        endDate: '9/30/16',
        oneYear: '18.00%',
        twoYear: '20.00%'
      },
      {
        Index: 17,
        startDate: '10/1/16',
        endDate: '9/30/17',
        oneYear: '18.00%',
        twoYear: '20.00%'
      },
      {
        Index: 18,
        startDate: '10/1/17',
        endDate: '9/30/18',
        oneYear: '19.25%',
        twoYear: '20.00%'
      },
      {
        Index: 19,
        startDate: '10/1/18',
        endDate: '6/13/19',
        oneYear: '19.00%',
        twoYear: '20.00%'
      },
      {
        Index: 20,
        startDate: '6/14/19',
        endDate: '9/30/19',
        oneYear: '1.50%',
        twoYear: '2.50%'
      },
      {
        Index: 21,
        startDate: '10/1/19',
        endDate: '9/30/20',
        oneYear: '1.50%',
        twoYear: '2.50%'
      }
    ],
    rgb: [
      {
        order: 33,
        startDate: '10/1/01',
        endDate: '9/30/02',
        oneYearRenewal: '4.00%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '6.00%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 34,
        startDate: '10/1/02',
        endDate: '9/30/03',
        oneYearRenewal: '2.00%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '4.00%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 35,
        startDate: '10/1/03',
        endDate: '9/30/04',
        oneYearRenewal: '4.50%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '7.50%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 36,
        startDate: '10/1/04',
        endDate: '9/30/05',
        oneYearRenewal: '3.50%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '6.50%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 37,
        startDate: '10/1/05',
        endDate: '9/30/06',
        oneYearRenewal: '2.75%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '5.50%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 38,
        startDate: '10/1/06',
        endDate: '9/30/07',
        oneYearRenewal: '4.25%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '7.25%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 39,
        startDate: '10/1/07',
        endDate: '9/30/08',
        oneYearRenewal: '3.00%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '5.75%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 40,
        startDate: '10/1/08',
        endDate: '9/30/09',
        oneYearRenewal: '4.50%',
        oneYearRenewalSub: 45,
        twoYearRenewal: '8.50%',
        twoYearRenewalSub: 85
      },
      {
        order: 41,
        startDate: '10/1/09',
        endDate: '9/30/10',
        oneYearRenewal: '3.00%',
        oneYearRenewalSub: 30,
        twoYearRenewal: '6.00%',
        twoYearRenewalSub: 60
      },
      {
        order: 42,
        startDate: '10/1/10',
        endDate: '9/30/11',
        oneYearRenewal: '2.25%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '4.50%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 43,
        startDate: '10/1/11',
        endDate: '9/30/12',
        oneYearRenewal: '3.75%',
        oneYearRenewalSub: 'N/A',
        twoYearRenewal: '7.25%',
        twoYearRenewalSub: 'N/A'
      },
      {
        order: 44,
        startDate: '10/1/12',
        endDate: '9/30/13',
        oneYearRenewal: '2.00%',
        oneYearRenewalSub: 20,
        twoYearRenewal: '4.00%',
        twoYearRenewalSub: 40
      },
      {
        order: 45,
        startDate: '10/1/13',
        endDate: '9/30/14',
        oneYearRenewal: '4.00%',
        oneYearRenewalSub: '',
        twoYearRenewal: '7.75%',
        twoYearRenewalSub: ''
      },
      {
        order: 46,
        startDate: '10/1/14',
        endDate: '9/30/15',
        oneYearRenewal: '1.00%',
        oneYearRenewalSub: '',
        twoYearRenewal: '2.75%',
        twoYearRenewalSub: ''
      },
      {
        order: 47,
        startDate: '10/1/15',
        endDate: '9/30/16',
        oneYearRenewal: '0.00%',
        oneYearRenewalSub: '',
        twoYearRenewal: '2.00%',
        twoYearRenewalSub: ''
      },
      {
        order: 48,
        startDate: '10/1/16',
        endDate: '9/30/17',
        oneYearRenewal: '0.00%',
        oneYearRenewalSub: '',
        twoYearRenewal: '2.00%',
        twoYearRenewalSub: ''
      },
      {
        order: 49,
        startDate: '10/1/17',
        endDate: '9/30/18',
        oneYearRenewal: '1.25%',
        oneYearRenewalSub: '',
        twoYearRenewal: '2.00%',
        twoYearRenewalSub: ''
      },
      {
        order: 50,
        startDate: '10/1/18',
        endDate: '9/30/19',
        oneYearRenewal: '1.50%',
        oneYearRenewalSub: '',
        twoYearRenewal: '2.50%',
        twoYearRenewalSub: ''
      },
      {
        order: 51,
        startDate: '10/1/19',
        endDate: '9/30/20',
        oneYearRenewal: '1.50%',
        oneYearRenewalSub: '',
        twoYearRenewal: '2.50%',
        twoYearRenewalSub: ''
      }
    ],
    tenant: {}
  }),
  methods: {
    addTenant () {
      this.tenants.push({})
    },
    percentage (percent, total) {
      return ((percent / 100) * total).toFixed(2)
    },
    calculateRenewal (index) {
      const tenant = this.tenants[index]
      const date = moment(tenant.date).format('M/D/YY')
      const item = this.rgb.find(r => {
        return (
          moment(date).isAfter(r.startDate) && moment(date).isBefore(r.endDate)
        )
      })
      if (item) {
        tenant.order = item.order
        return item.order
      } else {
        return 'N/A'
      }
    },
    renewalIncrease (index) {
      const tenant = this.tenants[index]

      if (tenant.order) {
        if (tenant.type !== 'r') {
          tenant.renewalIncrease = 0
          return 'N/A'
        }
        const item = this.rgb.find(r => r.order === tenant.order)
        if (tenant.year === 1) {
          tenant.renewalIncrease = item.oneYearRenewal
          if (tenant.lrr < 1000) return item.oneYearRenewalSub
          return item.oneYearRenewal
        } else {
          if (tenant.lrr < 1000) return item.oneYearRenewalSub
          tenant.renewalIncrease = item.twoYearRenewal
          return item.twoYearRenewal
        }
      }
    },
    vacancyRow (index) {
      const tenant = this.tenants[index]
      const date = moment(tenant.date).format('M/D/YY')
      const item = this.vacancyIncreases.find(r => {
        return (
          moment(date).isAfter(r.startDate) && moment(date).isBefore(r.endDate)
        )
      })
      if (item) {
        tenant.renewalIndex = item.Index
        return item.Index
      } else {
        return 'N/A'
      }
    },
    vacancyIncrease (index) {
      const tenant = this.tenants[index]
      const item = this.vacancyIncreases.find(
        r => r.Index === tenant.renewalIndex
      )
      if (tenant.type != 'v') {
        tenant.vacancyIncrease = 0
        return 'N/A'
      }
      if (tenant.type === 'v' && tenant.year === '1') {
        tenant.vacancyIncrease = item.oneYear
        return item.oneYear
      }
      if (tenant.type === 'v' && tenant.year === '2') {
        tenant.vacancyIncrease = item.twoYear
        return item.twoYear
      }
    },
    longitivityIncrease (index) {
      const tenant = this.tenants[index]
      if (tenant.type != 'v') {
        tenant.longitivityIncrease = 0
        return 'N/A'
      } else if (Number(tenant.longevity) >= 8) {
        tenant.longitivityIncrease = (Number(tenant.longevity) * 0.6).toFixed(2)
        return (Number(tenant.longevity) * 0.6).toFixed(2) + '%'
      }
    },
    legalIncreasePercent (index) {
      const tenant = this.tenants[index]
      let vi = tenant.vacancyIncrease
        ? Number(tenant.vacancyIncrease.split('%')[0])
        : 0
      let ri = tenant.renewalIncrease
        ? Number(tenant.renewalIncrease.split('%')[0])
        : 0
      let li = tenant.longitivityIncrease
        ? Number(tenant.longitivityIncrease)
        : 0
      tenant.legalIncrease = vi + ri + li
      return tenant.legalIncrease + '%'
    },
    legalIncreaseDolar (index) {
      const tenant = this.tenants[index]
      tenant.legalIncreaseDolar = Number(
        this.percentage(tenant.legalIncrease, tenant.lrr)
      )
      return tenant.legalIncreaseDolar + '$'
    },
    newLegalRent (index) {
      const tenant = this.tenants[index]
      tenant.legalIncreaseDolar = Number(
        this.percentage(tenant.legalIncrease, tenant.lrr)
      )
      tenant.newLegalRent = Number(tenant.lrr) + tenant.legalIncreaseDolar
      return Number(tenant.lrr) + tenant.legalIncreaseDolar + '$'
    },
    potentialOverCharge (index) {
      const tenant = this.tenants[index]
      tenant.potentialOverCharge = 0
      if (tenant.lrr > tenant.newLegalRent) {
        tenant.potentialOverCharge = tenant.lrr - tenant.newLegalRent
        return tenant.potentialOverCharge
      }
    },
    requiredIaiLT35 (index) {
      const tenant = this.tenants[index]
      const date = moment(tenant.date).format('M/D/YY')
      if (tenant.potentialOverCharge === 0) {
        tenant.requiredIailt35 = 0
        return 'N/A'
      } else if (date <= moment('06-14-19').format('M/D/YY')) {
        return tenant.potentialOverCharge * 40
      } else if (date >= moment('06-14-19').format('M/D/YY')) {
        return tenant.potentialOverCharge * 168
      }
    },
    requiredIaiGT35 (index) {
      const tenant = this.tenants[index]
      const date = moment(tenant.date).format('M/D/YY')
      if (tenant.potentialOverCharge === 0) {
        tenant.requiredIaiGt35 = 0
        return 'N/A'
      } else if (date <= moment('09-24-11').format('M/D/YY')) {
        return tenant.potentialOverCharge * 40
      } else if (
        date >= moment('09-24-11').format('M/D/YY') &&
        date <= moment('06-13-19').format('M/D/YY')
      ) {
        return tenant.potentialOverCharge * 60
      } else if (date >= moment('06-13-19').format('M/D/YY')) {
        return tenant.potentialOverCharge * 168
      }
    }
  }
}
</script>
