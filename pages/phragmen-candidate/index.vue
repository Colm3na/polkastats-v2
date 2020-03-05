<template>
  <index></index>
</template>
<!-- This is commented to fix temporally
<template>
  <div>
    <section>
      <b-container class="page-candidate main pt-3 pb-5">
        <template v-for="(candidate, index) in candidates">
          <template v-if="candidate.pub_key_stash == accountId">
            <div class="row">
              <div class="col-2 col-lg-1">
                <template v-if="index > 0">
                  <nuxt-link :to="{name: 'phragmen-candidate', query: { accountId: candidates[index-1].pub_key_stash } }" :title="'Previous candidate: ' + candidates[index-1].pub_key_stash">
                    <i class="fas fa-2x fa-chevron-left"></i>
                  </nuxt-link>
                </template>
              </div>
              <div class="col-8 col-lg-10 text-center">
                <h4 class="mb-1">Candidate {{ indexes[accountId] }}</h4>
              </div>
              <div class="col-2 col-lg-1 text-right">
                <template v-if="index < candidates.length - 1">
                  <nuxt-link :to="{name: 'phragmen-candidate', query: { accountId: candidates[index+1].pub_key_stash } }" :title="'Next candidate: ' + candidates[index+1].pub_key_stash">
                    <i class="fas fa-2x fa-chevron-right"></i>
                  </nuxt-link>  
                </template>
              </div>
            </div>
            <div class="candidate card mt-4 mb-3">
              <div class="card-body">
                <div class="row">
                  <div class="col-md-3 mb-2 text-center">

                    <div v-if="hasPolkStatsIdentity(candidate.pub_key_stash)">
                      <div v-if="getPolkStatsIdentity(candidate.pub_key_stash).logo !== ''">
                        <img v-bind:src="getPolkStatsIdentity(candidate.pub_key_stash).logo" class="identity mt-2" />
                        <h4 class="mt-2 mb-2" v-if="getPolkStatsIdentity(candidate.pub_key_stash).full_name !== ''">{{ getPolkStatsIdentity(candidate.pub_key_stash).full_name }}</h4>
                      </div>
                      <div v-else>
                        <Identicon :value="candidate.pub_key_stash" :size="80" :theme="'polkadot'" :key="candidate.pub_key_stashd" />
                      </div>
                    </div>
                    <div v-else>
                      <Identicon :value="candidate.pub_key_stash" :size="80" :theme="'polkadot'" :key="candidate.pub_key_stash" />
                    </div>
                    <p class="mt-3 mb-0 rank">
                      rank #{{ candidate.rank }}
                    </p>
                    <p class="bonded mb-0" v-b-tooltip.hover title="Total stake">{{ formatAmount(candidate.stake_total) }}</p>
                    <p class="mb-0">
                      <small>
                        <span v-b-tooltip.hover title="Self bonded">{{ formatAmount(candidate.stake_validator) }}</span>
                        <span v-b-tooltip.hover title="Bonded by nominators">(+{{ formatAmount(candidate.other_stake_sum) }})</span>
                      </small>
                    </p>
                  </div>
                  <div class="col-md-9">
                    <h4 class="card-title mb-4 account mt-4 mt-sm-0 mt-md-0 mt-lg-0 mt-xl-0">
                      <span v-if="hasNickname(candidate.pub_key_stash)">
                        {{ getNickname(candidate.pub_key_stash) }}
                      </span>
                      <span v-else>
                        {{ candidate.pub_key_stash }}
                      </span>
                    </h4>
                    <div class="row" v-if="candidate.pub_key_stash">
                      <div class="col-md-3 mb-1">
                        <strong>Stash</strong>
                      </div>
                      <div class="col-md-9 mb-1">
                        <Identicon :value="candidate.pub_key_stash" :size="20" :theme="'polkadot'" :key="candidate.pub_key_stash" />
                        <a v-bind:href="blockExplorer.account + candidate.pub_key_stash" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="candidate.pub_key_stash">{{ indexes[candidate.pub_key_stash] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[candidate.pub_key_stash] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="candidate.pub_key_controller">
                      <div class="col-md-3 mb-1">
                        <strong>Controller</strong>
                      </div>
                      <div class="col-md-9 mb-1">
                        <Identicon :value="candidate.pub_key_controller" :size="20" :theme="'polkadot'" :key="candidate.pub_key_controller" />
                        <a v-bind:href="blockExplorer.account + candidate.pub_key_controller" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="candidate.pub_key_controller">{{ indexes[candidate.pub_key_controller] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[candidate.pub_key_controller] }}</span>
                        </a>
                      </div>
                    </div>

                    <div v-if="hasIdentity(candidate.pub_key_stash)">
                      <div class="row" v-if="getIdentity(candidate.pub_key_stash).identity.hasOwnProperty('display')">
                        <div class="col-md-3 mb-1">
                          <strong>Name</strong>
                        </div>
                        <div class="col-md-9 mb-1 fee">
                          {{ getIdentity(candidate.pub_key_stash).identity.display }}
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(candidate.pub_key_stash).identity.hasOwnProperty('email')">
                        <div class="col-md-3 mb-2">
                          <strong>Email</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a :href="`mailto:${getIdentity(candidate.pub_key_stash).identity.email}`" target="_blank">
                            {{ getIdentity(candidate.pub_key_stash).identity.email }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(candidate.pub_key_stash).identity.hasOwnProperty('legal')">
                        <div class="col-md-3 mb-2">
                          <strong>Legal</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          {{ getIdentity(candidate.pub_key_stash).identity.legal }}
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(candidate.pub_key_stash).identity.hasOwnProperty('riot')">
                        <div class="col-md-3 mb-2">
                          <strong>Riot</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee"
                           <a :href="`https://riot.im/app/#/user/${getIdentity(candidate.pub_key_stash).identity.riot}`" target="_blank">
                            {{ getIdentity(candidate.pub_key_stash).identity.riot }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(candidate.pub_key_stash).hasOwnProperty('twitter')">
                        <div class="col-md-3 mb-2">
                          <strong>Twitter</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a :href="`https://twitter.com/${getIdentity(candidate.pub_key_stash).identity.twitter}`" target="_blank">
                            {{ getIdentity(candidate.pub_key_stash).identity.twitter }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(candidate.pub_key_stash).identity.hasOwnProperty('web')">
                        <div class="col-md-3 mb-2">
                          <strong>Web</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a v-bind:href="getIdentity(candidate.pub_key_stash).identity.web" target="_blank">
                            {{ getIdentity(candidate.pub_key_stash).identity.web }}
                          </a>
                        </div>
                      </div>
                    </div>

                    <div class="voters mt-2">
                      <template v-if="candidate.voters">
                        <template v-if="candidate.voters.length > 0">
                          <a class="" data-toggle="collapse" v-bind:href="'#candidate-voters-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'candidate-voters-' + index">
                            <h6 class="h6 candidates d-inline mr-4"><i class="fas"></i> Voters ({{ candidate.voters.length }})</h6>
                          </a>
                        </template>
                      </template>
                      <template v-if="candidate.voters">
                        <template v-if="candidate.voters.length > 0">
                          <div class="voters collapse pt-2 pb-3"  v-bind:id="'candidate-voters-' + index" v-bind:data-parent="'#candidate-voters-' + index">
                            <div v-for="(voter, index) in candidate.voters" class="row" v-bind:key="voter.pub_key_nominator">
                              <div class="col-8 mb-1 who">
                                <Identicon :value="voter.pub_key_nominator" :size="20" :theme="'polkadot'" :key="voter.pub_key_nominator" />                      
                                <a v-bind:href="blockExplorer.account + voter.pub_key_nominator" target="_blank">
                                  <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="voter.pub_key_nominator">{{ indexes[voter.pub_key_nominator] }}</span>
                                  <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ indexes[voter.pub_key_nominator] }}</span>                        
                                </a>
                              </div>
                              <div class="col-4 text-right value">
                                {{ formatAmount(voter.stake_nominator) }}
                              </div>
                            </div>
                          </div>
                        </template>
                      </template>
                    </div>
                  </div>
                </div>
              </div>
            </div>

          </template>
        </template>
      </b-container>
    </section>
  </div>
</template>
-->
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
      title: 'PolkaStats - Polkadot Kusama phragmen candidate ' + this.$route.query.accountId,
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama phragmen candidate ' + this.$route.query.accountId }
      ]
    }
  },
  mixins: [commonMixin],
  data: function() {
    return {
      accountId: this.$route.query.accountId,
      blockExplorer,
      backendBaseURL,
      polling: null
    }
  },
  computed: {
    candidates () {
      return this.$store.state.phragmen.info.candidates;
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

    // Force update of phragmen candidates list if empty
    if (this.$store.state.phragmen.info.candidates.length == 0) {
      vm.$store.dispatch('phragmen/update');
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
    if (this.$store.state.nicknames.list.length === 0) {
      vm.$store.dispatch('nicknames/update');
    }

    // Force update of account indexes list if empty
    if (this.$store.state.indexes.list.length == 0) {
      vm.$store.dispatch('indexes/update');
    }
    console.log('store', this.$store.state)
    // Update data every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('phragmen/update');
      vm.$store.dispatch('identities/update');
      vm.$store.dispatch('nicknames/update');
      vm.$store.dispatch('stakingIdentities/update');
    }, 10000);

    // Update account indexes every 1 min
    this.pollingIndexes = setInterval(() => {
      vm.$store.dispatch('indexes/update'); 
    }, 60000);
    
  },
  beforeDestroy: function () {
    clearInterval(this.polling);
    clearInterval(this.pollingIndexes);
  },
  methods: {
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
    getPolkStatsIdentity(stashId) {
      let filteredArray =  this.$store.state.identities.list.filter(obj => {
        return obj.stashId === stashId
      });
      return filteredArray[0];
    },
    hasPolkStatsIdentity(stashId) {
      return this.$store.state.identities.list.some(obj => {
        return obj.stashId === stashId;
      });
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
.page-candidate .favorite {
  position: absolute;
  top: 0.4rem;
  right: 0.4rem;
  cursor: initial;
}
.page-candidate .clipboard {
  display: inline-block;
}
</style>