<template>
  <div>
    <section>
      <b-container class="validator-page main pt-3 pb-5">
        <template v-for="(validator, index) in validators">
          <template v-if="validator.accountId === accountId">
            <div class="row">
              <div class="col-2 col-lg-1">
                <template v-if="index > 0">
                  <nuxt-link :to="{name: 'validator', query: { accountId: validators[index-1].accountId } }" :title="'Previous validator: ' + validators[index-1].accountId">
                    <i class="fas fa-2x fa-chevron-left"></i>
                  </nuxt-link>
                </template>
              </div>
              <div class="col-8 col-lg-10 text-center">
                <h4 class="mb-1">Validator {{ indexes[accountId] }}</h4>
              </div>
              <div class="col-2 col-lg-1 text-right">
                <template v-if="index < validators.length - 1">
                  <nuxt-link :to="{name: 'validator', query: { accountId: validators[index+1].accountId } }" :title="'Next validator: ' + validators[index+1].accountId">
                    <i class="fas fa-2x fa-chevron-right"></i>
                  </nuxt-link>  
                </template>
              </div>
            </div>
            <div class="validator-detail card mt-4 mb-3">
              <div class="card-body">
                <i v-if="validator.imOnline.isOnline" class="imOnline fas fa-check-circle" v-b-tooltip.hover v-bind:title="getImOnlineMessage(validator)"></i>
                <i v-else class="imOffline fas fa-times-circle" v-b-tooltip.hover v-bind:title="getImOnlineMessage(validator)"></i>
                <i v-if="validator.currentElected" class="elected fas fa-chevron-circle-right" v-b-tooltip.hover title="Elected for next session"></i>
                <i v-else class="notElected fas fa-times-circle" v-b-tooltip.hover title="Not elected for next session"></i>
                <i v-if="isFavorite(validator.accountId)" class="favorite fas fa-star" style="color: #f1bd23" v-b-tooltip.hover title="In Favorites"></i>
                <i v-else class="favorite fas fa-star" style="color: #e6dfdf;" v-b-tooltip.hover title="Not in Favorites"></i>
                <div class="row">
                  <div class="col-md-3 mb-2 text-center">
                    <div v-if="hasPolkaStatsIdentity(validator.stashId)">
                      <div v-if="getPolkaStatsIdentity(validator.accountId).logo">
                        <img v-bind:src="getPolkaStatsIdentity(validator.accountId).logo" class="img-fluid" style="max-width: 150px;" />
                        <h3 class="mt-2 mb-2" v-if="getPolkaStatsIdentity(validator.accountId).full_name">{{ getPolkaStatsIdentity(validator.accountId).full_name }}</h3>
                      </div>
                      <div v-else>
                        <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                      </div>
                    </div>
                    <div v-else>
                      <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                    </div>
                    <p class="mt-3 mb-0 rank">
                      rank #{{ index+1 }}
                    </p>
                    <template v-if="validator.stakers">
                      <p v-if="validator.stakers.total > 0" class="bonded mb-0" v-b-tooltip.hover title="Total bonded">{{ formatAmount(validator.stakers.total) }}</p>
                      <p v-else class="bonded mb-0" v-b-tooltip.hover title="Total bonded">{{ formatAmount(validator.stakingLedger.total) }}</p>
                      <p class="mb-0" v-if="validator.stakers.own !== validator.stakers.total">
                        <small>
                          <span v-b-tooltip.hover title="Self bonded" v-if="validator.stakers.own > 0">{{ formatAmount(validator.stakers.own) }}</span>
                          <span v-b-tooltip.hover title="Bonded by nominators" v-if="(validator.stakers.total - validator.stakers.own) > 0">(+{{ formatAmount(validator.stakers.total - validator.stakers.own) }})</span>
                        </small>
                      </p>
                      <p class="mb-0" v-b-tooltip.hover title="Percentage over total bonded stake">{{ getStakePercent(validator.stakers.total, totalStakeBonded) }}% of total stake</p>
                    </template>
                    <p class="mb-0" v-if="validator.currentEraPointsEarned">{{ validator.currentEraPointsEarned }} era points</p>
                  </div>
                  <div class="col-md-9">
                    <div class="row" v-if="hasNickname(validator.accountId)">
                      <div class="col-md-12">
                        <h4 class="card-title mb-4 account mt-4 mt-sm-0 mt-md-0 mt-lg-0 mt-xl-0">
                          {{ getNickname(validator.accountId) }}
                        </h4>
                      </div>
                    </div>
                    <div class="row" v-if="validator.accountId">
                      <div class="col-md-3 mb-2">
                        <strong>Stash</strong>
                      </div>
                      <div class="col-md-9 mb-2">
                        <Identicon :value="validator.accountId" :size="20" :theme="'polkadot'" :key="validator.accountId" />
                        <a v-bind:href="blockExplorer.account + validator.accountId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.accountId">{{ indexes[validator.accountId] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.accountId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="validator.controllerId">
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
                    <div class="row" v-if="validator.validatorPrefs.commission">
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

                    <!-- Stakers -->
                    <b-tabs content-class="mt-4 mb-2" class="mt-4">
                      <!-- Stakers -->
                      <b-tab v-if="validator.stakers.others.length > 0" :title="`Stakers (${validator.stakers.others.length})`" active>
                        <!-- Filter -->
                        <b-row class="mb-4">
                          <b-col lg="12">
                            <b-form-input
                              v-model="filter"
                              type="search"
                              id="filterInput"
                              placeholder="Search staker by account or account index"
                            ></b-form-input>
                          </b-col>
                        </b-row>
                        <!-- Mobile sorting -->
                        <div class="row d-block d-sm-block d-md-block d-lg-none d-xl-none">
                          <b-col lg="6" class="my-1">
                            <b-form-group
                              label="Sort"
                              label-cols-sm="3"
                              label-align-sm="right"
                              label-size="sm"
                              label-for="sortBySelect"
                              class="mb-4"
                            >
                              <b-input-group size="sm">
                                <b-form-select v-model="sortBy" id="sortBySelect" :options="sortOptions" class="w-75">
                                  <template v-slot:first>
                                    <option value="">-- none --</option>
                                  </template>
                                </b-form-select>
                                <b-form-select v-model="sortDesc" size="sm" :disabled="!sortBy" class="w-25">
                                  <option :value="false">Asc</option>
                                  <option :value="true">Desc</option>
                                </b-form-select>
                              </b-input-group>
                            </b-form-group>
                          </b-col>
                        </div>
                        <!-- Table with sorting and pagination-->
                        <div class="table-responsive">
                          <b-table
                            stacked="md"
                            id="nominators-table"
                            head-variant="dark"
                            :fields="fields"
                            :items="validatorStakers"
                            :per-page="perPage"
                            :current-page="currentPage"
                            :sort-by.sync="sortBy"
                            :sort-desc.sync="sortDesc"
                            :filter="filter"
                            :filterIncludedFields="filterOn"
                            @filtered="onFiltered"
                          >
                            <template slot="rank" slot-scope="data">
                              <p class="text-center mb-0">
                                {{ data.item.rank }}
                              </p>
                            </template>
                            <template slot="who" slot-scope="data">

                              <div class="d-block d-sm-block d-md-none d-lg-none d-xl-none text-center py-2">
                                <p class="mb-0">
                                  rank #{{ data.item.rank }}
                                </p>
                                <p class="mt-2 mb-0">
                                  <Identicon :value="data.item.who" :size="20" :theme="'polkadot'" :key="data.item.who" />
                                  <nuxt-link :to="{name: 'nominator', query: { accountId: data.item.who } }" title="Nominator details">
                                    <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="data.item.who">{{ indexes[data.item.who] }}</span>
                                    <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ indexes[data.item.who] }}</span>                        
                                  </nuxt-link>
                                </p>
                                <p class="mt-2 mb-0">
                                  {{ formatAmount(data.item.value) }} ({{ parseFloat(data.item.percent).toFixed(3) }} %)
                                </p>
                              </div>
                              <div class="d-none d-sm-none d-md-block d-lg-block d-xl-block">
                                <p class="mb-0">
                                  <Identicon :value="data.item.who" :size="20" :theme="'polkadot'" :key="data.item.who" />
                                  <nuxt-link :to="{name: 'nominator', query: { accountId: data.item.who } }" title="Nominator details">
                                    <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="data.item.who">{{ indexes[data.item.who] }}</span>
                                    <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ indexes[data.item.who] }}</span>                        
                                  </nuxt-link>
                                </p>
                              </div>

                            </template>
                            <template slot="percent" slot-scope="data">
                              <p class="text-right mb-0">
                                {{ parseFloat(data.item.percent).toFixed(3) }} %
                              </p>
                            </template>
                            <template slot="amountOrder" slot-scope="data">
                              <p class="text-right mb-0">
                                {{ formatAmount(data.item.value) }}
                              </p>
                            </template>
                          </b-table>
                          <b-pagination
                            v-model="currentPage"
                            :total-rows="totalRows"
                            :per-page="perPage"
                            aria-controls="nominators-table"
                          ></b-pagination>
                        </div>
                      </b-tab>
                      <!-- Current session ids -->
                      <b-tab v-if="validator.sessionIds.length > 0" :title="`Session ids (${validator.sessionIds.length})`">
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
                      </b-tab>
                      <!-- Next session ids -->
                      <b-tab v-if="validator.nextSessionIds.length > 0" :title="`Next session ids (${validator.nextSessionIds.length})`">
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
                      </b-tab>
                    </b-tabs>

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
import { backendBaseURL, blockExplorer } from '../../polkastats.config.js';
import commonMixin from '../../mixins/commonMixin.js';

export default {
  head () {
    return {
      title: 'PolkaStats - Polkadot Kusama validator ' + this.$route.query.accountId,
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama validator ' + this.$route.query.accountId }
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
      perPage: 10,
      currentPage: 1,
      sortBy: `rank`,
      sortDesc: false,
      filter: null,
      filterOn: [],
      totalRows: 1,
      fields: [
        { key: 'rank', label: 'Rank', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'who', label: 'Staker', sortable: true },
        { key: 'percent', label: 'Percentage', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'amountOrder', label: 'Amount', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` }
      ],
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
      },
      totalIssuance: ""
    }
  },
  computed: {
    validators() {
      if (this.$store.state.validators.list.length > 0) {
        let validator = this.$store.state.validators.list
          .find( validator => validator.accountId === this.accountId );
        this.totalRows = validator.stakers.others.length;
      }
      return this.$store.state.validators.list;
    },
    validatorStakers() {
      if (this.$store.state.validators.list.length > 0) {
        let validator = this.$store.state.validators.list
          .find( validator => validator.accountId === this.accountId );
        // console.log(validator);


        // Calculate others stake total amount
        let stakeTotal, stakeOwn, stakeOthers;

        if (isHex(validator.stakers.total)) {
          stakeTotal = new BN(validator.stakers.total.toString().substring(2, validator.stakers.total.length), 16);
        } else {
          stakeTotal = new BN(validator.stakers.total, 10);
        }
        
        if (isHex(validator.stakers.own)) {
          stakeOwn = new BN(validator.stakers.own.toString().substring(2, validator.stakers.own.length), 16);
        } else {
          stakeOwn = new BN(validator.stakers.own, 10);
        }
        stakeOthers = stakeTotal.sub(stakeOwn);



        let stakers = validator.stakers.others.slice();

        stakers.sort((a, b) => {
          if (a.value && b.value) {
            let stakeA, stakeB;

            // console.log(`a`, a.value);
            // console.log(`b`, b.value);

            if (isHex(a.value)) {
              stakeA = new BN(a.value.toString().substring(2, a.value.length), 16);
            } else {
              stakeA = new BN(a.value, 10);
            }
            
            if (isHex(b.value)) {
              stakeB = new BN(b.value.toString().substring(2, b.value.length), 16);
            } else {
              stakeB = new BN(b.value, 10);
            }

            // console.log(`a`, stakeA.toString(10));
            // console.log(`b`, stakeB.toString(10));
            // console.log(`res`, stakeA.lt(stakeB));
            
            return stakeA.lt(stakeB) ? 1 : -1;

            // return stakeA.cmp(stakeB);

          }
        });

        return stakers.map((staker, index) => {
          return {
            ...staker,
            rank: index+1,
            amountOrder: index+1,
            accountIndex: this.indexes[staker.who],
            percent: this.getStakePercent(staker.value, stakeOthers)
          }
        });
      }
    },
    identities() {
      return this.$store.state.identities.list;
    },
    nicknames() {
      return this.$store.state.nicknames.list;
    },
    indexes() {
      return this.$store.state.indexes.list;
    },
    totalStakeBonded () {
      return this.$store.state.validators.totalStakeBonded;
    },
    totalStakeBondedPercen() {
      if (this.totalStakeBonded !== 0 && this.totalIssuance !== "") {
        let totalIssuance = new BN(this.totalIssuance, 10);
        let totalStakeBonded = this.totalStakeBonded.mul(new BN('100000', 10));
        return totalStakeBonded.div(totalIssuance);
      } else {
        return 0;
      }
    },
    sortOptions() {
      // Create an options list from our fields
      return this.fields
        .filter(f => f.sortable)
        .map(f => {
          return { text: f.label, value: f.key }
        })
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
    if (this.$store.state.validators.list.length == 0) {
      vm.$store.dispatch('validators/update');
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

    // Update validators every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('validators/update');
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
      axios.get(`${this.backendBaseURL}/validator/graph/daily/${this.accountId}`)
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
      axios.get(`${this.backendBaseURL}/validator/graph/weekly/${this.accountId}`)
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
      axios.get(`${this.backendBaseURL}/validator/graph/monthly/${this.accountId}`)
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
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
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
.validator-detail .favorite {
  cursor: initial;
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
.validator-page .imOnline {
  position: absolute;
  top: 0.4rem;
  left: 0.4rem;
  font-size: 1.1rem;
  color: green;
}
.validator-page .imOffline {
  position: absolute;
  top: 0.4rem;
  left: 0.4rem;
  font-size: 1.1rem;
  color: red;
}
.validator-page .elected {
  position: absolute;
  top: 0.4rem;
  left: 2rem;
  font-size: 1.1rem;
  color: #2697e2;
}
.validator-page .notElected {
  position: absolute;
  top: 0.4rem;
  left: 2rem;
  font-size: 1.1rem;
  color: red;
}
.validator-page .favorite {
  position: absolute;
  top: 0.4rem;
  right: 0.4rem;
}
.validator-page #nominators-table th,
.validator-page #nominators-table td {
  padding: 0.5rem;
}
.validator-page #nominators-table th {
  text-align: center;
}
</style>