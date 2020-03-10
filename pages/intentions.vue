<template>
  <div>
    <section>
      <b-container class="main pt-4 page-intentions">
        <!-- Filter -->
        <b-row>
          <b-col lg="12" class="mb-4">
            <b-form-input
              v-model="filter"
              type="search"
              id="filterInput"
              placeholder="Search intention by account, account index, identity display name or keybase name"
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
            id="intentions-table"
            head-variant="dark"
            :fields="fields"
            :items="intentions"
            :per-page="perPage"
            :current-page="currentPage"
            :sort-by.sync="sortBy"
            :sort-desc.sync="sortDesc"
            :filter="filter"
            :filterIncludedFields="filterOn"
            @filtered="onFiltered"
          >
            <template slot="rank" slot-scope="data">
              <span class="text-right mb-0">
                {{ data.item.rank }}
              </span>
            </template>
            <template slot="accountId" slot-scope="data">
              <div class="d-block d-sm-block d-md-none d-lg-none d-xl-none text-center">
                <b-row>
                  <b-col cols="2" class="rank">
                    <span>#{{ data.item.rank }}</span>
                  </b-col>
                  <b-col cols="3" align-self="center">
                    <div v-if="hasIdentity(data.item.accountId)">
                      <div v-if="getIdentity(data.item.accountId).logo !== ''">
                        <img v-bind:src="getIdentity(data.item.accountId).logo" class="identity mt-2" />
                      </div>
                    </div>
                    <div v-else>
                      <Identicon :value="data.item.accountId" :size="80" :theme="'polkadot'" :key="data.item.accountId" />
                    </div>                
                  </b-col>
                  <b-col cols="7" align-self="center">          
                    <nuxt-link :to="{name: 'intention', query: { accountId: data.item.accountId } }" title="Intention details">
                      <h4 v-if="hasIdentity(data.item.accountId)" class="mt-2 mb-2">
                        {{ getIdentity(data.item.accountId).full_name }}
                      </h4>
                      <h4 v-else-if="hasKusamaIdentity(data.item.accountId)" class="mt-2 mb-2">
                        {{ hasKusamaIdentity(data.item.accountId).display }}
                      </h4>
                      <h4 v-else class="mt-2 mb-2">
                        <span class="d-inline d-sm-inline d-md-inline d-lg-inline d-xl-none">{{ indexes[data.item.accountId] }}</span>
                        <span class="d-none d-sm-none d-md-none d-lg-none d-xl-inline">{{ indexes[data.item.accountId] }}</span>
                      </h4>
                    </nuxt-link>
                  <div v-if="data.item.activeStake">
                    <p class="bonded mb-0" v-b-tooltip.hover title="Active bonded">
                      {{ formatAmount(data.item.activeStake) }}
                    </p>
                  </div>
                  <div v-if="data.item.totalStake">
                    <p class="mb-0">
                      <small>
                        <span v-b-tooltip.hover title="Total bonded">
                          {{ formatAmount(data.item.totalStake) }}
                        </span>
                      </small>
                    </p>
                  </div>
                  </b-col>
                  
                </b-row>
              </div>
              <div class="d-none d-sm-none d-md-block d-lg-block d-xl-block">
                <div v-if="hasIdentity(data.item.accountId)" class="d-inline-block">
                  <div v-if="getIdentity(data.item.accountId).logo !== ''" class="d-inline-block">
                    <img v-bind:src="getIdentity(data.item.accountId).logo" class="identity-small d-inline-block" />
                  </div>
                </div>
                <div v-else class="d-inline-block">
                  <Identicon :value="data.item.accountId" :size="20" :theme="'polkadot'" :key="data.item.accountId" />
                </div>
                <nuxt-link :to="{name: 'intention', query: { accountId: data.item.accountId } }" title="Intention details">
                  <span v-if="hasIdentity(data.item.accountId)">
                    {{ getIdentity(data.item.accountId).full_name }}
                  </span>
                  <span v-else-if="hasKusamaIdentity(data.item.accountId)">
                    {{ getKusamaIdentity(data.item.accountId).display }}
                  </span>
                  <span v-else>
                    <span class="d-inline d-sm-inline d-md-inline d-lg-inline d-xl-none">{{ indexes[data.item.accountId] }}</span>
                    <span class="d-none d-sm-none d-md-none d-lg-none d-xl-inline">{{ indexes[data.item.accountId] }}</span>
                  </span>
                </nuxt-link>
              </div>
            </template>
            <template slot="totalStake" slot-scope="data">
              <div v-if="data.item.totalStake">
                <p class="text-right mb-0">
                  {{ formatAmount(data.item.totalStake) }}
                </p>
              </div>
            </template> 
            <template slot="commission" slot-scope="data">
              <div v-if="typeof data.item.commission == 'number'">
                <p class="text-right mb-0">
                  {{ (data.item.commission / 10000000).toFixed(2) }}%
                </p>
              </div>
            </template>
            <template slot="favorite" slot-scope="data">
              <p class="text-center mb-0">
                <a class="favorite" v-on:click="toggleFavorite(data.item.accountId)">
                  <i v-if="data.item.favorite" class="fas fa-star" style="color: #f1bd23" v-b-tooltip.hover title="Remove from Favorites"></i>
                  <i v-else class="fas fa-star" style="color: #e6dfdf;" v-b-tooltip.hover title="Add to Favorites"></i>
                </a>
              </p>
            </template>
          </b-table>
          <b-pagination
            v-model="currentPage"
            :total-rows="totalRows"
            :per-page="perPage"
            aria-controls="intentions-table"
          ></b-pagination>
        </div>
      </b-container>
    </section>
  </div>
</template>
<script>
import { mapMutations } from 'vuex';
import axios from 'axios';
import bootstrap from 'bootstrap';
import Identicon from '../components/identicon.vue';
import Network from '../components/network.vue';
import { isHex } from '@polkadot/util';
import BN from 'bn.js';
import { blockExplorer } from '../polkastats.config.js';
import commonMixin from '../mixins/commonMixin.js';

export default {
  head () {
    return {
      title: 'PolkaStats -  Polkadot Kusama intention validators',
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama intention validators' }
      ]
    }
  },
  mixins: [commonMixin],
  data: function() {
    return {
      perPage: 10,
      currentPage: 1,
      sortBy: `rank`,
      sortDesc: false,
      filter: null,
      filterOn: [],
      totalRows: 1,
      fields: [
        { key: 'rank', label: '#', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'accountId', label: 'Intention', sortable: true },
        { key: 'totalStake', label: 'Total Stake', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'commission', label: 'Commission', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'favorite', label: '⭐', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` }
      ],
      blockExplorer,
      favorites: [],
      polling: null
    }
  },
  computed: {
    network () {
      return this.$store.state.network.info;
    },
    intentions () {
      let intentionsObject = [];
      for(let i = 0; i < this.$store.state.intentions.list.length; i++) {
        let intention = this.$store.state.intentions.list[i];

        let identity = "";
        if (this.hasIdentity(intention.accountId)) {
          identity = this.getIdentity(intention.accountId);
        }

        let kusamaIdentity = "";
        if (this.hasKusamaIdentity(intention.accountId)) {
          kusamaIdentity = this.hasKusamaIdentity(intention.accountId);
        }

        intentionsObject.push({
          rank: i+1,
          accountId: intention.accountId,
          accountIndex: this.indexes[intention.accountId],
          totalStake: intention.stakingLedger.total,
          activeStake: intention.stakingLedger.active,
          commission: intention.validatorPrefs.commission,
          favorite: this.isFavorite(intention.accountId),
          kusamaIdentity,
          identity
        });
      }
      return intentionsObject;
    },
    identities() {
      return this.$store.state.identities.list;
    },
    indexes() {
      return this.$store.state.indexes.list;
    },
    totalStakeBondedPercen() {
      if (this.totalStakeBonded !== 0 && this.network.totalIssuance !== "") {
        let totalIssuance = new BN(this.network.totalIssuance, 10);
        let totalStakeBonded = this.totalStakeBonded.mul(new BN('100', 10));
        return totalStakeBonded.div(totalIssuance);
      } else {
        return 0;
      }
    },
    totalStakeBonded () {
      return this.$store.state.validators.totalStakeBonded
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

    // Force update of network info
    vm.$store.dispatch('network/update');
    
    // Force update of intentions list if empty
    if (this.$store.state.intentions.list.length == 0) {
      vm.$store.dispatch('intentions/update');
    }
    this.totalRows = this.$store.state.intentions.list.length;

    // Force update of indentity list if empty
    if (this.$store.state.identities.list.length == 0) {
      vm.$store.dispatch('identities/update');
    }

    // Force update of staking identities list if empty
    if (this.$store.state.stakingIdentities.list.length === 0) {
      vm.$store.dispatch('stakingIdentities/update');
    }

    // Force update of indexes list if empty
    if (this.$store.state.indexes.list.length === 0) {
      vm.$store.dispatch('indexes/update');
    }

    // Update network info and intention validators every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('network/update');
      vm.$store.dispatch('intentions/update');
      vm.$store.dispatch('stakingIdentities/update');
      if (!this.filter) this.totalRows = this.$store.state.intentions.list.length;
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
    getRank(validator) {
      // Receives validator accountId
      for (var i=0; i < this.validators.length; i++) {
        if (this.validators[i].accountId == validator) {
          return i + 1;
        }
      }
      return false;
    },
    hasIdentity(stashId) {
      return this.$store.state.identities.list.some(obj => {
        return obj.stashId === stashId;
      });
    },
    getIdentity(stashId) {
      let filteredArray =  this.$store.state.identities.list.filter(obj => {
        return obj.stashId === stashId
      });
      console.log(filteredArray[0]);
      return filteredArray[0];
    },
    hasKusamaIdentity(stashId) {
      return this.$store.state.stakingIdentities.list.some(obj => {
        return obj.accountId === stashId;
      });
    },
    getKusamaIdentity(stashId) {
      let filteredArray =  this.$store.state.stakingIdentities.list.filter(obj => {
        return obj.accountId === stashId
      });
      console.log(filteredArray[0]);
      return filteredArray[0].identity;
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    }
  },
  watch: {
    favorites: function (val) {
      //console.log(val);
      this.$cookies.set('favorites', val, {
        path: '/',
        maxAge: 60 * 60 * 24 * 7
      });
    }
  },
  components: {
    Identicon,
    Network
  }
}
</script>
<style>
body {
  font-size: 0.9rem;
}
.page-intentions .table.b-table.b-table-stacked-md>tbody>tr>[data-label]>div {
  padding: 0;
}
.page-intentions .favorite {
  font-size: 1.1rem;
}
.validator .bg-offline {
  background-color: rgba(239, 57, 74, 0.12) !important;
}
.validator .bg-candidate {
  background-color: rgba(21, 240, 86, 0.12) !important;
}
.rank {
  font-size: 1.6rem;
  color: #7d7378;
}
.account {
  color: #670d35;
}
.bonded {
  color: #d75ea1;
  font-weight: 700;
  font-size: 1.3rem;
}
[data-toggle="collapse"] .fas:before {   
  content: "\f078";
}
[data-toggle="collapse"][aria-expanded="false"] .fas:before {
  content: "\f054";
}
.nominators {
  color: #670d35;
}
.nominator {
  font-size: 0.9rem;
}
.nominator .value {
  color: #d75ea1;
  font-weight: 700;
}
.fee, .unstake {
  color: #d75ea1;
  font-weight: 700;
}
.block {
  color: #d75ea1;
  font-weight: bold;
}
.block:hover {
  color: #d75ea1;
}
.tab-content .validator:nth-child(1) {
  border-top: 0;
}
.fas.fa-copy {
  cursor: pointer;
  color: #d75ea1;
  font-size: 0.9rem;
  margin-left: 0.1rem;
}
.identity {
  max-width: 80px;
}
#intentions-table th {
  text-align: center;
}
#intentions-table th:first-child {
  width: 8%;
}
#intentions-table th:nth-child(2) {
  width: 42%;
}
#intentions-table th:nth-child(3) {
  width: 20%;
}
#intentions-table th:nth-child(4) {
  width: 20%;
}
#intentions-table th:nth-child(5) {
  width: 10%;
}
#intentions-table .identicon {
  display: inline;
  margin-right: 0.2rem;
  cursor: copy;
}
#intentions-table .identicon div {
  display: inline;
}
.page-item.active .page-link {
    z-index: 1;
    color: #fff;
    background-color: #343a40;
    border-color: #343a40;
}
.page-link {
    position: relative;
    display: block;
    padding: 0.5rem 0.75rem;
    margin-left: -1px;
    line-height: 1.25;
    color: #343a40;
    background-color: #fff;
    border: 1px solid #dee2e6;
}
@media (max-width: 767px) {
  #intentions-table {
    background-color: transparent;
    padding: 0 0.5rem;
  }
  #intentions-table tr {
    border-radius: 0.8rem;
    box-shadow: 1px 1px 2px 2px #a2a6a8;
    padding: 1rem 0.5rem;
    margin: 1rem 0;
  }
  .rank {
    text-align: left;
    margin-top: -0.5rem;
  }
}
</style>
