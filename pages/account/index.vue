<template>
  <div>
    <section>
      <b-container class="account-page main pt-4 pb-5">
        
        <!-- Info message -->
        <b-alert show dismissible variant="success" class="text-center mb-4">
          Data is updated every 5 minutes
        </b-alert>

        <template v-for="(account, index) in accounts">
          <template v-if="account.accountId === accountId">
            <div class="row">
              <div class="col-2 col-lg-1">
                <template v-if="index > 0">
                  <nuxt-link :to="{name: 'account', query: { accountId: accounts[index-1].accountId } }" :title="'Previous account: ' + accounts[index-1].accountId">
                    <i class="fas fa-2x fa-chevron-left"></i>
                  </nuxt-link>
                </template>
              </div>
              <div class="col-8 col-lg-10 text-center">
                <h4 class="mb-1">Account {{ accountId }}</h4>
              </div>
              <div class="col-2 col-lg-1 text-right">
                <template v-if="index < accounts.length - 1">
                  <nuxt-link :to="{name: 'account', query: { accountId: accounts[index+1].accountId } }" :title="'Next account: ' + accounts[index+1].accountId">
                    <i class="fas fa-2x fa-chevron-right"></i>
                  </nuxt-link>  
                </template>
              </div>
            </div>
            <div class="card mt-4 mb-3">
              <div class="card-body">
                <p class="text-center mb-2">
                  <Identicon :value="account.accountId" :size="80" :theme="'polkadot'" :key="account.accountId"></Identicon>
                  <h4 class="text-center mb-4">{{ account.accountIndex }}</h4>
                  <h4 class="text-center mb-4 amount" v-b-tooltip.hover title="Free Balance">{{ formatAmount(account.balances.freeBalance) }}</h4>
                </p>
                <table class="table table-striped">
                  <tbody>
                    <tr>
                      <td>Account ID</td>
                      <td class="text-right">
                        <a v-bind:href="blockExplorer.account + account.accountId" target="_blank" class="d-block my-2">
                          <Identicon :value="account.accountId" :size="20" :theme="'polkadot'" :key="account.accountId" />
                          <span v-b-tooltip.hover title="See address in PolkaScan">{{ account.accountId }}</span>
                        </a>
                      </td>
                    </tr>
                    <tr>
                      <td>Account Index</td>
                      <td class="text-right">{{ account.accountIndex }}</td>
                    </tr>
                    <tr v-if="account.nickname">
                      <td>Nickname</td>
                      <td class="text-right">{{ account.nickname }}</td>
                    </tr>
                    <tr v-if="account.identity.display">
                      <td>Identity::display</td>
                      <td class="text-right">{{ account.identity.display }}</td>
                    </tr>
                    <tr v-if="account.identity.email">
                      <td>Identity::email</td>
                      <td class="text-right"><a :href="`mailto:${account.identity.email}`" target="_blank">{{ account.identity.email }}</a></td>
                    </tr>
                    <tr v-if="account.identity.legal">
                      <td>Identity::legal</td>
                      <td class="text-right">{{ account.identity.legal }}</td>
                    </tr>
                    <tr v-if="account.identity.riot">
                      <td>Identity::riot</td>
                      <td class="text-right">{{ account.identity.riot }}</td>
                    </tr>
                    <tr v-if="account.identity.web">
                      <td>Identity::web</td>
                      <td class="text-right"><a :href="account.identity.web" target="_blank">{{ account.identity.web }}</a></td>
                    </tr>
                    <tr v-if="account.identity.twitter">
                      <td>Identity::twitter</td>
                      <td class="text-right"><a :href="`https://twitter.com/${account.identity.twitter.substr(1, account.identity.twitter.length)}`" target="_blank">{{ account.identity.twitter }}</a></td>
                    </tr>
                    <tr v-if="account.identity.judgements">
                      <td>Identity::judgements</td>
                      <td class="text-right">
                        <span v-if="account.identity.judgements.length > 0">
                          {{ account.identity.judgements }}
                        </span>
                        <span>
                          No
                        </span>
                      </td>
                    </tr>
                    <tr>
                      <td>Account Nonce</td>
                      <td class="text-right">{{ account.balances.accountNonce }}</td>
                    </tr>
                    <tr>
                      <td>Free Balance</td>
                      <td class="text-right">{{ formatAmount(account.balances.freeBalance) }}</td>
                    </tr>
                    <tr>
                      <td>Available Balance</td>
                      <td class="text-right">{{ formatAmount(account.balances.availableBalance) }}</td>
                    </tr>
                    <tr>
                      <td>Locked Balance</td>
                      <td class="text-right">{{ formatAmount(account.balances.lockedBalance) }}</td>
                    </tr>
                    <tr>
                      <td>Locked Breakdowns</td>
                      <td class="text-right">
                        <p class="mb-0" v-if="account.balances.lockedBreakdown.length === 0">
                          No
                        </p>
                        <p v-else class="mb-1" v-for="(locked, index) in account.balances.lockedBreakdown" v-bind:key="locked.id">
                          <strong>{{ formatAmount(locked.amount) }}</strong> until <strong>{{ formatNumber(locked.until) }}</strong>
                        </p>
                      </td>
                    </tr>
                    <tr>
                      <td>Reserved Balance</td>
                      <td class="text-right">{{ formatAmount(account.balances.reservedBalance) }}</td>
                    </tr>
                    <tr>
                      <td>Is Vesting?</td>
                      <td class="text-right">{{ account.balances.isVesting ? `Yes` : `No` }}</td>
                    </tr>
                    <tr>
                      <td>Vested Balance</td>
                      <td class="text-right">{{ formatAmount(account.balances.vestedBalance) }}</td>
                    </tr>
                    <tr>
                      <td>Vesting Total</td>
                      <td class="text-right">{{ formatAmount(account.balances.vestingTotal) }}</td>
                    </tr>
                    <tr>
                      <td>Voting Balance</td>
                      <td class="text-right">{{ formatAmount(account.balances.votingBalance) }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </template>
        </template>
      </b-container>
    </section>
  </div>
</template>
<script>
import { mapMutations } from 'vuex';
import Identicon from '../../components/identicon.vue';
import { blockExplorer } from '../../polkastats.config.js';
import commonMixin from '../../mixins/commonMixin.js';

export default {
  head () {
    return {
      title: 'PolkaStats - Kusama account ' + this.$route.query.accountId,
      meta: [
        { hid: 'description', name: 'description', content: 'Kusama account ' + this.$route.query.accountId }
      ]
    }
  },
  mixins: [commonMixin],
  data: function() {
    return {
      accountId: this.$route.query.accountId,
      blockExplorer,
      polling: null
    }
  },
  computed: {
    accounts() {
      return this.$store.state.accounts.list;
    }
  },
  created: function () {
    var vm = this;
    
    // Force update of account list if empty
    if (this.$store.state.accounts.list.length == 0) {
      vm.$store.dispatch('accounts/update');
    }
    this.totalRows = this.$store.state.accounts.list.length;

    // Update data every 5 minutes
    this.polling = setInterval(() => {
      vm.$store.dispatch('accounts/update');
      if (!this.filter) this.totalRows = this.$store.state.accounts.list.length;
    }, 300000);

  },
  beforeDestroy: function () {
    clearInterval(this.polling);
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
.identicon {
  cursor: pointer;
}
.account-page .table tr td:first-child {
  width: 30%;
  font-weight: bold;
}
.account-page .table tr td:nth-child(2) {
  width: 70%;
}
.account-page .table tr td .identicon {
  display: inline-block;
}
.account-page .amount {
  color: #ef1073;
  font-weight: 700;
}
</style>