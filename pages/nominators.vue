<template>
  <div>
    <section>
      <b-container class="main pt-4">
        <!-- Filter -->
        <b-row>
          <b-col lg="12" class="mb-4">
            <b-form-input
              v-model="filter"
              type="search"
              id="filterInput"
              placeholder="Search nominator by account, account index or identity display name"
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
            :items="nominators"
            :per-page="perPage"
            :current-page="currentPage"
            :sort-by.sync="sortBy"
            :sort-desc.sync="sortDesc"
            :filter="filter"
            :filterIncludedFields="filterOn"
            @filtered="onFiltered"
          >
            <template slot="rank" slot-scope="data">
              {{ data.item.rank }}
            </template>
            <template slot="accountId" slot-scope="data">
              <div class="d-block d-sm-block d-md-none d-lg-none d-xl-none text-center">
                <b-row>
                  <b-col cols="2" class="rank">
                    <span>#{{ data.item.rank }}</span>
                  </b-col>
                  <b-col cols="3" align-self="center">
                    <Identicon :value="data.item.accountId" :size="80" :theme="'polkadot'" :key="data.item.accountId" />
                  </b-col>
                  <b-col cols="7" align-self="center">
                    <nuxt-link :to="{name: 'nominator', query: { accountId: data.item.accountId } }" title="Nominator details">
                      <h4 v-if="hasKusamaIdentity(data.item.accountId)">
                        {{ getKusamaIdentity(data.item.accountId).display }}
                      </h4>
                      <h4 v-else>
                        <span class="d-inline d-sm-inline d-md-inline d-lg-inline d-xl-none">{{ indexes[data.item.accountId] }}</span>
                        <span class="d-none d-sm-none d-md-none d-lg-none d-xl-inline">{{ indexes[data.item.accountId] }}</span>
                      </h4>
                    </nuxt-link>
                    <p class="mb-0">
                      {{ formatAmount(data.item.totalStake) }}
                    </p>
                  </b-col>              
                </b-row>
              </div>
              <div class="d-none d-sm-none d-md-block d-lg-block d-xl-block">
                <Identicon :value="data.item.accountId" :size="20" :theme="'polkadot'" :key="data.item.accountId" />
                <nuxt-link :to="{name: 'nominator', query: { accountId: data.item.accountId } }" title="Nominator details">
                  <span v-if="hasKusamaIdentity(data.item.accountId)">
                    {{ getKusamaIdentity(data.item.accountId).display }}
                  </span>
                  <span v-else>
                    <span class="d-inline d-sm-inline d-md-inline d-lg-inline d-xl-none">{{ indexes[data.item.accountId] }}</span>
                    <span class="d-none d-sm-none d-md-none d-lg-none d-xl-inline">{{ indexes[data.item.accountId] }}</span>
                  </span>
                </nuxt-link>
              </div>
            </template>
            <template slot="nominations" slot-scope="data">
              <p class="text-right mb-0">
                {{ data.item.nominations }}
              </p>
            </template>
            <template slot="totalStake" slot-scope="data">
              <p class="text-right mb-0">
                {{ formatAmount(data.item.totalStake) }}
              </p>
            </template>
            <template slot="favorite" slot-scope="data">
              <p class="text-center mb-0">
                <a class="favorite" v-on:click="toggleFavorite(data.item.accountIndex)">
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
            aria-controls="validators-table"
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
      title: 'PolkaStats - Polkadot Kusama nominators',
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama nominators' }
      ]
    }
  },
  mixins: [commonMixin],
  data: function() {
    return {
      blockExplorer,
      polling: null,
      perPage: 10,
      currentPage: 1,
      sortBy: ``,
      sortDesc: false,
      filter: null,
      filterOn: [],
      totalRows: 1,
      fields: [
        { key: 'rank', label: 'Rank', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'accountId', label: 'Nominator', sortable: true },
        { key: 'nominations', label: 'Nominations', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'totalStake', label: 'Total stake', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'favorite', label: '⭐', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` }
      ],
      favorites: []
    }
  },
  computed: {
    validators () {
      return this.$store.state.validators.list;
    },
    identities() {
      return this.$store.state.identities.list;
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
            const accountIndex = this.indexes[nominator.who];
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
              nominatorTmp[0].nominations++;
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

              let kusamaIdentity = "";
              if (this.hasKusamaIdentity(nominator.who)) {
                kusamaIdentity = this.hasKusamaIdentity(nominator.who);
              }

              nominatorStaking.push({
                accountId: nominator.who,
                accountIndex,
                kusamaIdentity,
                totalStake: bn,
                nominations: 1,
                staking: [{
                  validator: validator.accountId,
                  amount: nominator.value
                }],
                favorite: this.isFavorite(accountIndex)
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
      return nominatorStaking;
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

    // Force update of validators list if empty
    if (this.$store.state.validators.list.length === 0) {
      vm.$store.dispatch('validators/update');
    }

    // Force update of indentities list if empty
    if (this.$store.state.identities.list.length === 0) {
      vm.$store.dispatch('identities/update');
    }

    // Force update of staking identities list if empty
    if (this.$store.state.stakingIdentities.list.length === 0) {
      vm.$store.dispatch('stakingIdentities/update');
    }

    // Force update of account indexes list if empty
    if (this.$store.state.indexes.list.length == 0) {
      vm.$store.dispatch('indexes/update');
    }

    // Update validators and staking identities every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('validators/update');
      vm.$store.dispatch('stakingIdentities/update');
    }, 10000);

    // Update PolkaStats identities and account indexes every 1 min
    this.pollingIndexes = setInterval(() => {
      vm.$store.dispatch('identities/update');
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
        this.favorites.push({ accountId: validator, name: 'Edit phragmen name...'});
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
        if (this.favorites[i].accountId === validator) {
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
#nominators-table .identicon {
  display: inline;
  margin-right: 0.2rem;
  cursor: copy;
}
#nominators-table .identicon div {
  display: inline;
}
@media (max-width: 767px) {
  #nominators-table {
    background-color: transparent;
    padding: 0 0.5rem;
  }
  #nominators-table tr {
    border-radius: 0.8rem;
    box-shadow: 1px 1px 2px 2px #a2a6a8;
    padding: 1rem 0.5rem;
    margin: 1rem 0;
  }
  .rank {
    text-align: left;
    margin-top: -0.5rem;
    margin-left: -0.5rem;
  }
}
</style>
