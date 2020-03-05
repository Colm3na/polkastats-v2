<template>
  <div>
    <section>
      <b-container class="nominator-page main pt-3 pb-5">
        <template v-for="(nominator, index) in nominators">
          <template v-if="nominator.accountId === accountId">
            <div class="row">
              <div class="col-2 col-lg-1">
                <template v-if="index > 0">
                  <nuxt-link :to="{name: 'nominator', query: { accountId: nominators[index-1].accountId } }" :title="'Previous nominator: ' + nominators[index-1].accountId">
                    <i class="fas fa-2x fa-chevron-left"></i>
                  </nuxt-link>
                </template>
              </div>
              <div class="col-8 col-lg-10 text-center">
                <h4 class="mb-1">Nominator {{ indexes[accountId] }}</h4>
              </div>
              <div class="col-2 col-lg-1 text-right">
                <template v-if="index < nominators.length - 1">
                  <nuxt-link :to="{name: 'nominator', query: { accountId: nominators[index+1].accountId } }" :title="'Next validator: ' + nominators[index+1].accountId">
                    <i class="fas fa-2x fa-chevron-right"></i>
                  </nuxt-link>  
                </template>
              </div>
            </div>           
            <div class="card mt-4 mb-3">
              <div class="card-body text-center">
                <Identicon :value="nominator.accountId" :size="80" :theme="'polkadot'" :key="nominator.accountId" />
                <a v-bind:href="blockExplorer.account + nominator.accountId" target="_blank" class="d-block my-2">
                  Nominator <span v-b-tooltip.hover title="See address in PolkaScan">{{ indexes[nominator.accountId] }}</span>
                </a>
                <p class="amount" v-b-tooltip.hover title="Total bonded">{{ formatDot(getTotalStake(nominator.staking)) }}</p>
                <h5>{{ nominator.staking.length }} nomination<span v-if="nominator.staking.length > 1">s</span>:</h5>
                <hr>          
                <!-- identity start -->
                    <div v-if="hasIdentity(nominator.accountId)">
                      <div class="row" v-if="getIdentity(nominator.accountId).identity.hasOwnProperty('display')">
                        <div class="col-md-3 mb-1">
                          <strong>Name</strong>
                        </div>
                        <div class="col-md-9 mb-1 fee">
                          {{ getIdentity(nominator.accountId).identity.display }}
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(nominator.accountId).identity.hasOwnProperty('email')">
                        <div class="col-md-3 mb-2">
                          <strong>Email</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                           <a :href="`mailto:${getIdentity(nominator.accountId).identity.email}`" target="_blank">
                            {{ getIdentity(nominator.accountId).identity.email }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(nominator.accountId).identity.hasOwnProperty('legal')">
                        <div class="col-md-3 mb-2">
                          <strong>Legal</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          {{ getIdentity(nominator.accountId).identity.legal }}
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(nominator.accountId).identity.hasOwnProperty('riot')">
                        <div class="col-md-3 mb-2">
                          <strong>Riot</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                           <a :href="`https://riot.im/app/#/user/${getIdentity(nominator.accountId).identity.riot}`" target="_blank">
                            {{ getIdentity(nominator.accountId).identity.riot }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(nominator.accountId).identity.hasOwnProperty('twitter')">
                        <div class="col-md-3 mb-2">
                          <strong>Twitter</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a :href="`https://twitter.com/${getIdentity(nominator.accountId).identity.twitter}`" target="_blank">
                            {{ getIdentity(nominator.accountId).identity.twitter }}
                          </a>
                        </div>
                      </div>
                      <div class="row" v-if="getIdentity(nominator.accountId).identity.hasOwnProperty('web')">
                        <div class="col-md-3 mb-2">
                          <strong>Web</strong>
                        </div>
                        <div class="col-md-9 mb-2 fee">
                          <a v-bind:href="getIdentity(nominator.accountId).identity.web" target="_blank">
                            {{ getIdentity(nominator.accountId).identity.web }}
                          </a>
                        </div>
                      </div>
                    </div>
                    <!-- identity end -->
                <div class="row">
                  <div class="col-6 col-md-4 col-lg-3 col-xl-2" v-for="nomination in nominator.staking" :key="nomination.validator">
                    <Identicon :value="nomination.validator" :size="40" :theme="'polkadot'" :key="nomination.validator" />
                    <nuxt-link v-if="hasNickname(nomination.validator)" :to="{name: 'validator', query: { accountId: nomination.validator } }" title="Validator details" class="mt-2 mb-0 d-block">
                      {{ getNickname(nomination.validator) }}
                    </nuxt-link>
                    <nuxt-link v-else :to="{name: 'validator', query: { accountId: nomination.validator } }" title="Validator details" class="mt-2 mb-0 d-block">
                      <span v-b-tooltip.hover v-bind:title="nomination.validator">{{ indexes[nomination.validator] }}</span>
                    </nuxt-link>
                    <p class="mt-0 mb-0">rank #{{ getRank(nomination.validator) }}</p>
                    <p class="mt-0 mb-2">
                      commission {{ (validators[getIndex(nomination.validator)].validatorPrefs.commission / 10000000).toFixed(2) }}%
                    </p>
                    <p class="amount">{{ formatDot(nomination.amount) }} <small>({{ (getTotalStakePercen(nominator.staking, nomination.amount) / 100).toFixed(2) }}%)</small></p>
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
<script>
import { mapMutations } from 'vuex'
import axios from 'axios';
import moment from 'moment';
import VueApexCharts from 'vue-apexcharts';
import Identicon from '../../components/identicon.vue';
import { formatBalance, isHex } from '@polkadot/util';
import BN from 'bn.js';
import { decimals, unit, backendBaseURL, blockExplorer } from '../../polkastats.config.js';

formatBalance.setDefaults({ decimals, unit });

export default {
  head () {
    return {
      title: 'PolkaStats - Kusama nominator ' + this.$route.query.accountId,
      meta: [
        { hid: 'description', name: 'description', content: 'Kusama nominator ' + this.$route.query.accountId }
      ]
    }
  },
  data: function() {
    return {
      accountId: this.$route.query.accountId,
      blockExplorer,
      polling: null
    }
  },
  computed: {
    validators () {
      return this.$store.state.validators.list;
    },
    identities() {
      return this.$store.state.identities.list;
    },
    nicknames() {
      return this.$store.state.nicknames.list;
    },
    indexes() {
      return this.$store.state.indexes.list
    },
    nominators () {
      let nominatorStaking = [];
      for(let i = 0; i < this.validators.length; i++) {
        let validator = this.validators[i];     
        if (validator.stakers.others.length > 0) {
          for (let j = 0; j < validator.stakers.others.length; j++) {
            let nominator = validator.stakers.others[j];
            if (nominatorStaking.find(nom => nom.accountId === nominator.who)) {
              let nominatorTmp = nominatorStaking.filter(nom => {
                return nom.accountId === nominator.who
              })
              let bn;
              if (isHex(nominator.value)) {
                bn = new BN(nominator.value.substring(2, nominator.value.length), 16);
              } else {
                bn = new BN(nominator.value.toString(), 10);
              }
              nominatorTmp[0].totalStake = nominatorTmp[0].totalStake.add(bn);
              nominatorTmp[0].staking.push({
                validator: validator.accountId,
                amount: nominator.value
              });
            } else {
              let bn;
              if (isHex(nominator.value)) {
                bn = new BN(nominator.value.substring(2, nominator.value.length), 16);
              } else {
                bn = new BN(nominator.value.toString(), 10);
              }
              nominatorStaking.push({
                accountId: nominator.who,
                totalStake: bn,
                staking: [{
                  validator: validator.accountId,
                  amount: nominator.value
                }]
              })
            }
          }
        }
      }
      nominatorStaking.sort(function compare( a, b ) {
        if ( a.totalStake.lt(b.totalStake) ){
          return 1;
        }
        if ( a.totalStake.gt(b.totalStake) ){
          return -1;
        }
        return 0;
      });
      nominatorStaking.map((nominator, index) => {
        nominator.rank = index+1;
      });
      this.totalRows = nominatorStaking.length;
      // console.log(nominatorStaking);
      return nominatorStaking;
    }
  },
  created: function () {
    var vm = this;
    
    // Force update of validators list if empty
    if (this.$store.state.validators.list.length === 0) {
      vm.$store.dispatch('validators/update');
    }

    // Force update of staking_identity list if empty
    if (this.$store.state.stakingIdentities.list.length == 0) {
      vm.$store.dispatch('stakingIdentities/update');
    }

    // Force update of indentities list if empty
    if (this.$store.state.identities.list.length === 0) {
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

    // Update validators, identities and nicknames every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('validators/update');
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
    formatNumber(n) {
      if (isHex(n)) {
        return (parseInt(n, 16).toString()).replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,');
      } else {
        return (n.toString()).replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,');
      }
    },
    formatDot(amount) {
      let bn;
      if (isHex(amount)) {
        bn = new BN(amount.substring(2, amount.length), 16);
      } else {
        bn = new BN(amount.toString());
      }
      return formatBalance(bn.toString(10));
    },  
    shortAddress(address) {
      return (address).substring(0,5) + ' .... ' + (address).substring(address.length - 5);
    },
    thousandsSeparator(n) {
      return n.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    getIndex(validator) {
      // Receives validator accountId
      for (var i=0; i < this.validators.length; i++) {
        if (this.validators[i].accountId === validator) {
          return i;
        }
      }
      return false;
    },
    getRank(validator) {
      // Receives validator accountId
      for (var i=0; i < this.validators.length; i++) {
        if (this.validators[i].accountId == validator) {
          return i + 1;
        }
      }
      return false;
    },
    makeToast(content = '', title = '', variant = null, solid = false) {
      this.$bvToast.toast(content, {
        title: title,
        variant: variant,
        solid: solid
      })
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
    getTotalStake(stake) {
      let totalStake = new BN('0', 10)
      if (stake.length > 0) {
        for(let i = 0; i < stake.length; i++) {
          let nomination = stake[i];
          let bn;
          if (isHex(nomination.amount)) {
            bn = new BN(nomination.amount.substring(2, nomination.amount.length), 16);
          } else {
            bn = new BN(nomination.amount.toString(), 10);
          }
          totalStake = totalStake.add(bn);
        }
        return totalStake;
      } else {
        return 0;
      }
    },
    getTotalStakePercen(stake, amount) {

      let totalStake = this.getTotalStake(stake);

      if (totalStake && amount) {
        let bn;
        if (isHex(amount)) {
          bn = new BN(amount.substring(2, amount.length), 16);
        } else {
          bn = new BN(amount.toString(), 10);
        }
        bn = bn.mul(new BN('10000', 10));
        return bn.div(totalStake);
      } else {
        return 0;
      }
    },
  },
  watch: {
    $route () {
      this.accountId = this.$route.query.accountId;
    }
  },
  components: {
    Identicon
  }
}
</script>
<style>
.amount {
  color: #ef1073;
  font-weight: 700;
  font-size: 1rem;
}
.identicon {
  cursor: pointer;
}
</style>