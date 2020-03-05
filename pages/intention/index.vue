<template>
  <div>
    <section>
      <b-container class="page-intention main pt-3 pb-5">
        <template v-for="(validator, index) in intentions">
          <template v-if="validator.accountId == accountId">
            <div class="row">
              <div class="col-2 col-lg-1">
                <template v-if="index > 0">
                  <nuxt-link :to="{name: 'intention', query: { accountId: intentions[index-1].accountId } }" :title="'Previous intention: ' + intentions[index-1].accountId">
                    <i class="fas fa-2x fa-chevron-left"></i>
                  </nuxt-link>
                </template>
              </div>
              <div class="col-8 col-lg-10 text-center">
                <h4 class="mb-1">Intention {{ indexes[accountId] }}</h4>
              </div>
              <div class="col-2 col-lg-1 text-right">
                <template v-if="index < intentions.length - 1">
                  <nuxt-link :to="{name: 'intention', query: { accountId: intentions[index+1].accountId } }" :title="'Next intention: ' + intentions[index+1].accountId">
                    <i class="fas fa-2x fa-chevron-right"></i>
                  </nuxt-link>  
                </template>
              </div>
            </div>
            <div class="validator-detail card mt-4 mb-3">
              <div class="card-body">
                <i v-if="isFavorite(validator.accountId)" class="favorite fas fa-star" style="color: #f1bd23" v-b-tooltip.hover title="In Favorites"></i>
                <i v-else class="favorite fas fa-star" style="color: #e6dfdf;" v-b-tooltip.hover title="Not in Favorites"></i>
                <div class="row">
                  <div class="col-md-3 mb-2 text-center">
                    <div v-if="hasPolkaStatsIdentity(validator.accountId)">
                      <div v-if="getPolkaStatsIdentity(validator.accountId).logo !== ''">
                        <img v-bind:src="getPolkaStatsIdentity(validator.accountId).logo" class="img-fluid" style="max-width: 150px;" />
                        <h3 class="mt-2 mb-2" v-if="getPolkaStatsIdentity(validator.accountId).full_name !== ''">{{ getPolkaStatsIdentity(validator.accountId).full_name }}</h3>
                      </div>
                      <div v-else>
                        <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                      </div>
                    </div>
                    <div v-else>
                      <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                    </div>
                    <p class="mb-0 rank">
                      rank #{{ index+1 }}
                    </p>
                    <p class="bonded mb-0" v-b-tooltip.hover title="Active bonded">
                      {{ formatAmount(validator.stakingLedger.active) }}
                    </p>
                    <p class="mb-0">
                      <small>
                        <span v-b-tooltip.hover title="Total bonded">
                          {{ formatAmount(validator.stakingLedger.total) }}
                        </span>
                      </small>
                    </p>
                  </div>
                  <div class="col-md-9">
                    <div class="row" v-if="hasNickname(validator.accountId)">
                      <div class="col-md-12">
                        <h4 class="card-title mb-4 account mt-4 mt-sm-0 mt-md-0 mt-lg-0 mt-xl-0">
                          {{ getNickname(validator.accountId) }}
                        </h4>
                      </div>
                    </div>
                    <div class="row">
                      <div class="col-md-3 mb-2">
                        <strong>Stash</strong>
                      </div>
                      <div class="col-md-9 mb-2">
                        <Identicon :value="validator.stashId" :size="20" :theme="'polkadot'" :key="validator.stashId" />
                        <a v-bind:href="blockExplorer.account + validator.stashId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.stashId">{{ indexes[validator.stashId] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.stashId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row">
                      <div class="col-md-3 mb-2">
                        <strong>Controller</strong>
                      </div>
                      <div class="col-md-9 mb-2">
                        <Identicon :value="validator.controllerId" :size="20" :theme="'polkadot'" :key="validator.controllerId" />
                        <a v-bind:href="blockExplorer.account + validator.controllerId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.controllerId">{{ indexes[validator.controllerId] }} </span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.controllerId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="validator.sessionIdHex">
                      <div class="col-md-3 mb-2">
                        <strong>Session id</strong>
                      </div>
                      <div id="session-id-info" class="col-md-9 mb-2">
                        <a class="" data-toggle="collapse" v-bind:href="'#sessionIdHex'" role="button" aria-expanded="false" v-bind:aria-controls="'sessionIdHex'">
                          <i class="fas"></i> {{ shortSessionId(validator.sessionIdHex) }}
                        </a>
                        <div class="collapse pt-2 pb-3"  v-bind:id="'sessionIdHex'" v-bind:data-parent="'#session-id-info'">
                          {{ validator.sessionIdHex }}
                        </div>
                      </div>
                    </div>
                    <div class="row" v-if="validator.nextSessionIdHex">
                      <div class="col-md-3 mb-2">
                        <strong>Next session id</strong>
                      </div>
                      <div id="next-session-id-info" class="col-md-9 mb-2">
                        <a class="" data-toggle="collapse" v-bind:href="'#nextSessionIdHex'" role="button" aria-expanded="false" v-bind:aria-controls="'nextSessionIdHex'">
                          <i class="fas"></i> {{ shortSessionId(validator.nextSessionIdHex) }}
                        </a>
                        <div class="collapse pt-2 pb-3"  v-bind:id="'nextSessionIdHex'" v-bind:data-parent="'#next-session-id-info'">
                          {{ validator.nextSessionIdHex }}
                        </div>
                      </div>
                    </div>
                    <div class="row" v-if="typeof validator.validatorPrefs.commission == 'number'">
                      <div class="col-md-3 mb-2">
                        <strong>Commission</strong>
                      </div>
                      <div class="col-md-9 mb-2 fee">
                        {{ (validator.validatorPrefs.commission / 10000000).toFixed(2) }}%
                      </div>
                    </div>
                    <div class="row mb-2">
                      <div class="col-md-3 mb-2">
                        <strong>Reward destination</strong>
                      </div>
                      <div class="col-md-9 mb-2 fee">
                        {{ formatRewardDest(validator.rewardDestination) }}
                      </div>
                    </div>
                    <!-- identity start -->
                    <div v-if="hasIdentity(validator.accountId)">
                      <div class="row" v-if="getIdentity(validator.accountId).identity.hasOwnProperty('display')">
                        <div class="col-md-3 mb-1">
                          <strong>Name</strong>
                        </div>
                        <div class="col-md-9 mb-1 fee">
                          {{ getIdentity(validator.accountId).identity.display }}
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(validator.accountId).identity.hasOwnProperty('email')">
                        <div class="col-md-3 mb-2">
                          <strong>Email</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a :href="`mailto:${getIdentity(validator.accountId).identity.email}`" target="_blank">
                            {{ getIdentity(validator.accountId).identity.email }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(validator.accountId).identity.hasOwnProperty('legal')">
                        <div class="col-md-3 mb-2">
                          <strong>Legal</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          {{ getIdentity(validator.accountId).identity.legal }}
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(validator.accountId).identity.hasOwnProperty('riot')">
                        <div class="col-md-3 mb-2">
                          <strong>Riot</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                           <a :href="`https://riot.im/app/#/user/${getIdentity(validator.accountId).identity.riot}`" target="_blank">
                            {{ getIdentity(validator.accountId).identity.riot }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(validator.accountId).identity.hasOwnProperty('twitter')">
                        <div class="col-md-3 mb-2">
                          <strong>Twitter</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a :href="`https://twitter.com/${getIdentity(validator.accountId).identity.twitter}`" target="_blank">
                            {{ getIdentity(validator.accountId).identity.twitter }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(validator.accountId).identity.hasOwnProperty('web')">
                        <div class="col-md-3 mb-2">
                          <strong>Web</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a v-bind:href="getIdentity(validator.accountId).identity.web" target="_blank">
                            {{ getIdentity(validator.accountId).identity.web }}
                          </a>
                        </div>
                      </div>
                    </div>
                    <!-- identity end -->
                    <div v-bind:id="'validator-info-' + index" class="mt-2">
                      <template v-if="validator.stakers.others.length > 0">
                        <a class="" data-toggle="collapse" v-bind:href="'#staker' + index" role="button" aria-expanded="false" v-bind:aria-controls="'staker' + index">
                          <h6 class="h6 nominators d-inline mr-4"><i class="fas"></i> Stakers ({{ validator.stakers.others.length }})</h6>
                        </a>
                      </template>
                      <template v-if="validator.sessionIds.length > 0">
                        <a class="" data-toggle="collapse" v-bind:href="'#current-session-id-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'current-session-id-' + index">
                          <h6 class="h6 nominators d-inline mr-4"><i class="fas"></i> Current session ids ({{ validator.sessionIds.length }})</h6>
                        </a>
                      </template>
                      <template v-if="validator.nextSessionIds.length > 0">
                        <a class="" data-toggle="collapse" v-bind:href="'#session-id-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'session-id-' + index">
                          <h6 class="h6 nominators d-inline mr-4"><i class="fas"></i> Next session ids ({{ validator.nextSessionIds.length }})</h6>
                        </a>
                      </template>
                      <template v-if="validator.sessionIds.length > 0">
                        <div class="nominator collapse pt-2 pb-3"  v-bind:id="'current-session-id-' + index" v-bind:data-parent="'#validator-info-' + index">
                          <div v-for="(sessionId, index) in validator.sessionIds" class="row" v-bind:key="index">
                            <div class="col-12 mb-1 who">
                              {{ index+1 }}.
                              <Identicon :value="sessionId" :size="20" :theme="'polkadot'" :key="sessionId" />                      
                              <a v-bind:href="blockExplorer.account + sessionId" target="_blank">
                                <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="sessionId">{{ shortAddress(sessionId) }}</span>
                                <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ sessionId }}</span>                        
                              </a>
                            </div>
                          </div>
                        </div>
                      </template>
                      <template v-if="validator.nextSessionIds.length > 0">
                        <div class="nominator collapse pt-2 pb-3"  v-bind:id="'session-id-' + index" v-bind:data-parent="'#validator-info-' + index">
                          <div v-for="(sessionId, index) in validator.nextSessionIds" class="row" v-bind:key="index">
                            <div class="col-12 mb-1 who">
                              {{ index+1 }}.
                              <Identicon :value="sessionId" :size="20" :theme="'polkadot'" :key="sessionId" />                      
                              <a v-bind:href="blockExplorer.account + sessionId" target="_blank">
                                <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="sessionId">{{ shortAddress(sessionId) }}</span>
                                <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ sessionId }}</span>                        
                              </a>
                            </div>
                          </div>
                        </div>
                      </template>
                      <template v-if="validator.stakers.others.length > 0">
                        <div class="nominator collapse pt-2 pb-3"  v-bind:id="'staker' + index" v-bind:data-parent="'#validator-info-' + index">
                          <div v-for="(staker, index) in validator.stakers.others" class="row" v-bind:key="index">
                            <div class="col-8 mb-1 who">
                              <Identicon :value="staker.who" :size="20" :theme="'polkadot'" :key="staker.who" />                      
                              <a v-bind:href="blockExplorer.account + staker.who" target="_blank">
                                <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="staker.who">{{ indexes[staker.who] }}</span>
                                <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ indexes[staker.who] }}</span>                        
                              </a>
                            </div>
                            <div class="col-4 text-right value">
                              {{ formatAmount(staker.value) }}
                            </div>
                          </div>
                        </div>
                      </template>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </template>
        </template>
        <div class="mt-5 text-center" id="stake-evolution-monthly-chart">
          <h3>Total bonded - Monthly chart <small class="change text-success ml-3" v-if="monthly.last - monthly.first > 0"><i class="far fa-thumbs-up"></i> +{{ formatAmount(monthly.last - monthly.first) }}</small><small class="change text-danger ml-3" v-if="monthly.last - monthly.first < 0"><i class="far fa-thumbs-down"></i> {{ formatAmount(monthly.last - monthly.first) }}</small></h3>
          <apexchart type=line height=350 :options="StakeEvolutionMonthlyChartOptions" :series="StakeEvolutionMonthlySeries" />
        </div>
        <div class="mt-5 mb-5 text-center" id="stake-evolution-weekly-chart">
          <h3>Total bonded - Weekly chart <small class="change text-success ml-3" v-if="weekly.last - weekly.first > 0"><i class="far fa-thumbs-up"></i> +{{ formatAmount(weekly.last - weekly.first) }}</small><small class="change text-danger ml-3" v-if="weekly.last - weekly.first < 0"><i class="far fa-thumbs-down"></i> {{ formatAmount(weekly.last - weekly.first) }}</small></h3>
          <apexchart type=line height=350 :options="StakeEvolutionWeeklyChartOptions" :series="StakeEvolutionWeeklySeries" />
        </div>
        <div class="mb-5 text-center" id="stake-evolution-daily-chart">
          <h3>Total bonded - Daily chart <small class="change text-success ml-3" v-if="daily.last - daily.first > 0"><i class="far fa-thumbs-up"></i> +{{ formatAmount(daily.last - daily.first) }}</small><small class="change text-danger ml-3" v-if="daily.last - daily.first < 0"><i class="far fa-thumbs-down"></i> {{ formatAmount(daily.last - daily.first) }}</small></h3>
          <apexchart type=line height=350 :options="StakeEvolutionDailyChartOptions" :series="StakeEvolutionDailySeries" />
        </div>
      </b-container>
    </section>
  </div>
</template>
<script>
import { mapMutations } from 'vuex'
import axios from 'axios';
import moment from 'moment';
import VueApexCharts from 'vue-apexcharts';
import Identicon from '../../components/identicon.vue';
import { isHex } from '@polkadot/util';
import BN from 'bn.js';
import { backendBaseURL, blockExplorer} from '../../polkastats.config.js';
import commonMixin from '../../mixins/commonMixin.js';

export default {
  head () {
    return {
      title: 'PolkaStats - Polkadot Kusama intention validator ' + this.$route.query.accountId,
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama intention validator ' + this.$route.query.accountId }
      ]
    }
  },
  mixins: [commonMixin],
  data: function() {
    return {
      accountId: this.$route.query.accountId,
      blockExplorer,
      backendBaseURL,
      polling: null,
      graphPolling: null,
      favorites: [],
      daily:{
        last: 0,
        first: 0
      },
      weekly:{
        last: 0,
        first: 0
      },
      monthly:{
        last: 0,
        first: 0
      },
      StakeEvolutionDailySeries: [{
          name: "Total bonded (KSM)",
          data: []
      }],
       StakeEvolutionWeeklySeries: [{
          name: "Total bonded (KSM)",
          data: []
      }],
       StakeEvolutionMonthlySeries: [{
          name: "Total bonded (KSM)",
          data: []
      }],
      StakeEvolutionDailyChartOptions: {
        chart: {
          height: 500,
          stacked: false,
          zoom: {
            enabled: false
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: 'straight'
        },
        markers: {
          size: 6
        },
        colors: ['#d75ea1'],
        grid: {
          row: {
            colors: ['#f3f3f3', 'transparent'], // takes an array which will be repeated on columns
            opacity: 0.5
          },
        },
        xaxis: {
          categories: [],
          type: 'datetime',
          title: {
            text: 'Date / time (UTC)'
          },          
          labels: {
            formatter: function (val) {
              return moment.unix(val).format('MM/DD/YYYY HH:mm');
            }
          },
          tooltip: {
            enabled: false
          }
        },
        yaxis: {
          title: {
            text: 'Total bonded (KSM)'
          },
          labels: {
            formatter: function (val) {
              return val;
            }
          }       
        }
      },
      StakeEvolutionWeeklyChartOptions: {
        chart: {
          height: 500,
          stacked: false,
          zoom: {
            enabled: false
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: 'straight'
        },
        markers: {
          size: 6
        },
        colors: ['#d75ea1'],
        grid: {
          row: {
            colors: ['#f3f3f3', 'transparent'], // takes an array which will be repeated on columns
            opacity: 0.5
          },
        },
        xaxis: {
          categories: [],
          type: 'datetime',
          title: {
            text: 'Date / time (UTC)'
          },          
          labels: {
            formatter: function (val) {
              return moment.unix(val).format('MM/DD/YYYY HH:mm');
            }
          },
          tooltip: {
            enabled: false
          }
        },
        yaxis: {
          title: {
            text: 'Total bonded (KSM)'
          },
          labels: {
            formatter: function (val) {
              return val;
            }
          }       
        }
      },
      StakeEvolutionMonthlyChartOptions: {
        chart: {
          height: 500,
          stacked: false,
          zoom: {
            enabled: false
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: 'straight'
        },
        markers: {
          size: 6
        },
        colors: ['#d75ea1'],
        grid: {
          row: {
            colors: ['#f3f3f3', 'transparent'], // takes an array which will be repeated on columns
            opacity: 0.5
          },
        },
        xaxis: {
          categories: [],
          type: 'datetime',
          title: {
            text: 'Date / time (UTC)'
          },          
          labels: {
            formatter: function (val) {
              return moment.unix(val).format('MM/DD/YYYY HH:mm');
            }
          },
          tooltip: {
            enabled: false
          }
        },
        yaxis: {
          title: {
            text: 'Total bonded (KSM)'
          },
          labels: {
            formatter: function (val) {
              return val;
            }
          }       
        }
      }
    }
  },
  computed: {
    intentions() {
      return this.$store.state.intentions.list
    },
    identities() {
      return this.$store.state.identities.list
    },
    nicknames() {
      return this.$store.state.nicknames.list
    },
    indexes() {
      return this.$store.state.indexes.list
    }
  },
  created: function () {
    var vm = this;

    // Get favorites from cookie
    if (this.$cookies.get('favorites')) {
      this.favorites = this.$cookies.get('favorites');
    }

    // Load graph data first time
    this.getValidatorDailyGraphData();
    this.getValidatorWeeklyGraphData();
    this.getValidatorMonthlyGraphData();

    // Force update of validators list if empty
    if (this.$store.state.intentions.list.length == 0) {
      vm.$store.dispatch('intentions/update');
    }

    // Force update of staking_identity list if empty
    if (this.$store.state.stakingIdentities.list.length == 0) {
      vm.$store.dispatch('stakingIdentities/update');
    }

    // Force update of indentity list if empty
    if (this.$store.state.identities.list.length == 0) {
      vm.$store.dispatch('identities/update');
    }

    // Force update of nicknames list if empty
    if (this.$store.state.nicknames.list.length == 0) {
      vm.$store.dispatch('nicknames/update');
    }

    // Force update of account indexes list if empty
    if (this.$store.state.indexes.list.length == 0) {
      vm.$store.dispatch('indexes/update');
    }

    // Update intention validators every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('intentions/update');
      vm.$store.dispatch('stakingIdentities/update');
    }, 10000);
    
    // Refresh graph data and account indexes every minute
    this.graphPolling = setInterval(() => {
      this.getValidatorDailyGraphData();
      this.getValidatorWeeklyGraphData();
      this.getValidatorMonthlyGraphData();
      vm.$store.dispatch('indexes/update');
    }, 60000);
    
  },
  beforeDestroy: function () {
    clearInterval(this.polling);
    clearInterval(this.graphPolling);
  },
  methods: {
    getValidatorDailyGraphData: function () {
      var vm = this;
      axios.get(`${this.backendBaseURL}/intention/graph/daily/${this.accountId}`)
        .then(function (response) {

          // Update chart data
          var newCategories = [];
          var newData = [];
      
          //console.log(response.data);

          for (var i = 0; i < response.data.length; i++) {
            // Insert firt point, last point and points with different values
            if (i == 0 || (i == response.data.length -1 ) || (i > 0 && response.data[i].amount != response.data[i-1].amount)) {

              // Save first and last point
              if (i == 0) vm.daily.last = response.data[i].amount;
              if (i == response.data.length -1) vm.daily.first = response.data[i].amount;

              newCategories.push(moment.unix(response.data[i].timestamp, 'YYYY-MM-DD HH:mm:ss.SSSSSS Z').format('YYYY-MM-DD HH:mm:ss'));
              newData.push(response.data[i].amount);
            }
          }
          
          newCategories.reverse();
          newData.reverse();
          
          // Make sure to update the whole options config and not just a single property to allow the Vue watch catch the change.
          vm.StakeEvolutionDailyChartOptions = {
            ...vm.StakeEvolutionDailyChartOptions,
            ...{
              xaxis: {
                categories: newCategories,
                type: 'datetime',
                title: {
                  text: 'Date time (UTC)'
                },        
                labels: {
                  formatter: function (val) {
                    return moment.unix(val/1000).format('MM/DD/YYYY HH:mm');
                  }
                }
              },
              yaxis: {
                title: {
                  text: 'Total bonded (KSM)'
                },
                labels: {
                  formatter: function (val) {
                    return (val / 1000000000000).toFixed(6);
                  }
                }         
              }
            }
          };
          
          // In the same way, update the series option
          vm.StakeEvolutionDailySeries = [{
            data: newData
          }];
          
        })
    },
    getValidatorWeeklyGraphData: function () {
      var vm = this;
      axios.get(`${this.backendBaseURL}/intention/graph/weekly/${this.accountId}`)
        .then(function (response) {

          // Update chart data
          var newCategories = [];
          var newData = [];
      
          //console.log(response.data);

          for (var i = 0; i < response.data.length; i++) {
            // Save first and last point
            if (i == 0) vm.weekly.last = response.data[i].amount;
            if (i == response.data.length -1) vm.weekly.first = response.data[i].amount;

            newCategories.push(moment.unix(response.data[i].timestamp, 'YYYY-MM-DD HH:mm:ss.SSSSSS Z').format('YYYY-MM-DD HH:mm:ss'));
            newData.push(response.data[i].amount);
          }
          
          newCategories.reverse();
          newData.reverse();
          
          // Make sure to update the whole options config and not just a single property to allow the Vue watch catch the change.
          vm.StakeEvolutionWeeklyChartOptions = {
            ...vm.StakeEvolutionWeeklyChartOptions,
            ...{
              xaxis: {
                categories: newCategories,
                type: 'datetime',
                title: {
                  text: 'Date time (UTC)'
                },        
                labels: {
                  formatter: function (val) {
                    return moment.unix(val/1000).format('MM/DD/YYYY HH:mm');
                  }
                }
              },
              yaxis: {
                title: {
                  text: 'Total bonded (KSM)'
                },
                labels: {
                  formatter: function (val) {
                    return (val / 1000000000000).toFixed(6);
                  }
                }         
              }
            }
          };
          
          // In the same way, update the series option
          vm.StakeEvolutionWeeklySeries = [{
            data: newData
          }];
          
        })
    },
    getValidatorMonthlyGraphData: function () {
      var vm = this;
      axios.get(`${this.backendBaseURL}/intention/graph/monthly/${this.accountId}`)
        .then(function (response) {

          // Update chart data
          var newCategories = [];
          var newData = [];
      
          //console.log(response.data);

          for (var i = 0; i < response.data.length; i++) {
            // Save first and last point
            if (i == 0) vm.monthly.last = response.data[i].amount;
            if (i == response.data.length -1) vm.monthly.first = response.data[i].amount;

            newCategories.push(moment.unix(response.data[i].timestamp, 'YYYY-MM-DD HH:mm:ss.SSSSSS Z').format('YYYY-MM-DD HH:mm:ss'));
            newData.push(response.data[i].amount);
          }
          
          newCategories.reverse();
          newData.reverse();
          
          // Make sure to update the whole options config and not just a single property to allow the Vue watch catch the change.
          vm.StakeEvolutionMonthlyChartOptions = {
            ...vm.StakeEvolutionMonthlyChartOptions,
            ...{
              xaxis: {
                categories: newCategories,
                type: 'datetime',
                title: {
                  text: 'Date time (UTC)'
                },        
                labels: {
                  formatter: function (val) {
                    return moment.unix(val/1000).format('MM/DD/YYYY HH:mm');
                  }
                }
              },
              yaxis: {
                title: {
                  text: 'Total bonded (KSM)'
                },
                labels: {
                  formatter: function (val) {
                    return (val / 1000000000000).toFixed(6);
                  }
                }         
              }
            }
          };
          
          // In the same way, update the series option
          vm.StakeEvolutionMonthlySeries = [{
            data: newData
          }];
          
        })
    },
    toggleFavorite(validator) {
      // Receives validator accountId
      if (this.isFavorite(validator)) {
        this.favorites.splice(this.getIndex(validator), 1);
      } else {
        this.favorites.push({ accountId: validator, name: 'Edit validator name...'});
      }
      return true;
    },
    isFavorite(validator) {
      // Receives validator accountId
      for (var i=0; i < this.favorites.length; i++) {
        if (this.favorites[i].accountId == validator) {
          return true;
        }
      }
      return false;
    },
    getIndex(validator) {
      // Receives validator accountId
      for (var i=0; i < this.favorites.length; i++) {
        if (this.favorites[i].accountId == validator) {
          return i;
        }
      }
      return false;
    },
    getIdentity(stashId) {
      let filteredArray =  this.$store.state.stakingIdentities.list.filter(obj => {
        return obj.accountId === stashId
      });
      return filteredArray[0];
    },
    hasIdentity(stashId) {
      return this.$store.state.stakingIdentities.list.some(obj => {
        return obj.accountId === stashId;
      });
    },
    hasPolkaStatsIdentity(stashId) {
      return this.$store.state.identities.list.some(obj => {
        return obj.stashId === stashId;
      });
    },
    getPolkaStatsIdentity(stashId) {
      let filteredArray =  this.$store.state.identities.list.filter(obj => {
        return obj.stashId === stashId
      });
      return filteredArray[0];
    },
    hasNickname(accountId) {
      return this.$store.state.nicknames.list.some(obj => {
        return obj.accountId === accountId;
      });
    },
    getNickname(accountId) {
      let filteredArray =  this.$store.state.nicknames.list.filter(obj => {
        return obj.accountId === accountId
      });
      return filteredArray[0].nickname;
    }
  },
  watch: {
    $route () {
      this.accountId = this.$route.query.accountId;

      // Update graph data
      this.getValidatorDailyGraphData();
      this.getValidatorWeeklyGraphData();
      this.getValidatorMonthlyGraphData();
      
      // Restart graph data polling
      clearInterval(this.graphPolling);
      this.graphPolling = setInterval(() => {
        this.getValidatorDailyGraphData();
        this.getValidatorWeeklyGraphData();
        this.getValidatorMonthlyGraphData();
      }, 60000);
    }
  },
  components: {
    apexchart: VueApexCharts,
    Identicon
  }
}
</script>
<style>
.fas.fa-copy {
  cursor: pointer;
  color: #d75ea1;
  font-size: 0.9rem;
  margin-left: 0.1rem;
}
.rank {
  font-size: 1.4rem;
  color: #7d7378;
}
.change {
  vertical-align: middle;
}
.validator-detail .col-md-9 .identicon {
  display: inline;
  margin-right: 0.2rem;
  cursor: copy;
}
.validator-detail .col-md-9 .identicon div {
  display: inline;
}
.page-intention .favorite {
  position: absolute;
  top: 0.4rem;
  right: 0.4rem;
  cursor: initial;
}
</style>