<template>
  <div>
    <section>
      <b-container class="page-phragmen main pt-4">
        <h1 class="text-center mb-4">Predicted candidates by phragmen election algorithm</h1>
        <!-- Phragmen alert I -->
        <b-alert show dismissible variant="primary" class="text-center">
          <p class="mt-3">We get validador slots and minimum validator count from the local kusama node using @polkadot/api and then run offline-phragmen with that params every 1 minute. Output is stored in a MySQL database and served by PolkaStats backend (<a href="https://polkastats.io:8443/phragmen" target="_blank">see raw json</a>).</p>
          <p>
            We use a modified version of offline-phragmen by <a href="https://github.com/kianenigma" target="_blank">kianenigma</a> (<a href="https://github.com/kianenigma/offline-phragmen" target="_blank">https://github.com/kianenigma/offline-phragmen</a>).
            The modification was just change the output to json, taking most of the code from <a href="https://github.com/soc1c/offline-phragmen" target="_blank">https://github.com/soc1c/offline-phragmen</a> by <a href="https://github.com/soc1c" target="_blank">soc1c</a>.
            Modified source is available here: <a href="https://github.com/mariopino/offline-phragmen" target="_blank">https://github.com/mariopino/offline-phragmen</a></p>     
        </b-alert>
        <!-- Phragmen alert II -->
        <b-alert show dismissible variant="warning" class="text-center">
          Calculated over {{ validator_count }} validator candidates, {{ nominator_count }} nominators and total issuance of {{ formatAmount(total_issuance) }}     
        </b-alert>
        <!-- Filter -->
        <b-row>
          <b-col lg="12" class="mb-4">
            <b-form-input
              v-model="filter"
              type="search"
              id="filterInput"
              placeholder="Search candidate by account, account index, identity display name or keybase name"
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
        <div>
          <b-table
            stacked="md"
            id="candidates-table"
            head-variant="dark"
            :fields="fields"
            :items="candidates"
            :per-page="perPage"
            :current-page="currentPage"
            :sort-by.sync="sortBy"
            :sort-desc.sync="sortDesc"
            :filter="filter"
            :filterIncludedFields="filterOn"
            @filtered="onFiltered"
          >
            <template slot="rank" slot-scope="data">
              <p class="text-right mb-0">
                {{ data.item.rank }}
              </p>
            </template>
            <template slot="pub_key_stash" slot-scope="data">
              <div class="d-block d-sm-block d-md-none d-lg-none d-xl-none text-center">
                 <a class="favorite" v-on:click="toggleFavorite(data.item.accountId)">
                  <i v-if="data.item.favorite" class="fas fa-star" style="color: #f1bd23" v-b-tooltip.hover title="Remove from Favorites"></i>
                  <i v-else class="fas fa-star" style="color: #e6dfdf;" v-b-tooltip.hover title="Add to Favorites"></i>
                </a>
                <div v-if="hasIdentity(data.item.pub_key_stash)">
                  <div v-if="getIdentity(data.item.pub_key_stash).logo !== ''">
                    <img v-bind:src="getIdentity(data.item.pub_key_stash).logo" class="identity mt-2" />
                  </div>
                </div>
                <div v-else>
                  <Identicon :value="data.item.pub_key_stash" :size="80" :theme="'polkadot'" :key="data.item.pub_key_stash" />
                </div>
                <nuxt-link :to="{name: 'phragmen-candidate', query: { accountId: data.item.pub_key_stash } }" title="Candidate details">
                  <h4 v-if="hasIdentity(data.item.pub_key_stash)" class="mt-2 mb-2">
                    {{ getIdentity(data.item.pub_key_stash).full_name }}
                  </h4>
                  <h4 v-else-if="hasKusamaIdentity(data.item.pub_key_stash)" class="mt-2 mb-2">
                    {{ hasKusamaIdentity(data.item.pub_key_stash).display }}
                  </h4>
                  <h4 v-else class="mt-2 mb-2">
                    <span class="d-inline d-sm-inline d-md-inline d-lg-inline d-xl-none">{{ indexes[data.item.pub_key_stash] }}</span>
                    <span class="d-none d-sm-none d-md-none d-lg-none d-xl-inline">{{ indexes[data.item.pub_key_stash] }}</span>
                  </h4>
                </nuxt-link>
                <p class="mt-2 mb-2 rank">
                  rank #{{ data.item.rank }}
                </p>
                <p class="bonded mb-0" v-b-tooltip.hover title="Total stake">{{ formatAmount(data.item.stake_total) }}</p>
                <p class="mb-0">
                  <small>
                    <span v-b-tooltip.hover title="Self bonded">{{ formatAmount(data.item.stake_validator) }}</span>
                    <span v-b-tooltip.hover title="Bonded by nominators">(+{{ formatAmount(data.item.other_stake_sum) }})</span>
                  </small>
                </p>
              </div>
              <div class="d-none d-sm-none d-md-block d-lg-block d-xl-block">
                <div v-if="hasIdentity(data.item.pub_key_stash)" class="d-inline-block">
                  <div v-if="getIdentity(data.item.pub_key_stash).logo !== ''" class="d-inline-block">
                    <img v-bind:src="getIdentity(data.item.pub_key_stash).logo" class="identity-small d-inline-block" />
                  </div>
                </div>
                <div v-else class="d-inline-block">
                  <Identicon :value="data.item.pub_key_stash" :size="20" :theme="'polkadot'" :key="data.item.pub_key_stash" />
                </div>
                <nuxt-link :to="{name: 'phragmen-candidate', query: { accountId: data.item.pub_key_stash } }" title="Candidate details">
                  <span v-if="hasIdentity(data.item.pub_key_stash)">
                    {{ getIdentity(data.item.pub_key_stash).full_name }}
                  </span>
                  <span v-else-if="hasKusamaIdentity(data.item.pub_key_stash)">
                    {{ getKusamaIdentity(data.item.pub_key_stash).display }}
                  </span>
                  <span v-else>
                    <span class="d-inline d-sm-inline d-md-inline d-lg-inline d-xl-none">{{ indexes[data.item.pub_key_stash] }}</span>
                    <span class="d-none d-sm-none d-md-none d-lg-none d-xl-inline">{{ indexes[data.item.pub_key_stash] }}</span>
                  </span>
                </nuxt-link>
              </div>
            </template>
            <template slot="other_stake_count" slot-scope="data">
              <p class="text-right mb-0">
                {{ data.item.other_stake_count }}
              </p>
            </template>
            <template slot="stake_validator" slot-scope="data">
              <p class="text-right mb-0">
                {{ formatAmount(data.item.stake_validator) }}
              </p>
            </template>
            <template slot="other_stake_sum" slot-scope="data">
              <p class="text-right mb-0">
                {{ formatAmount(data.item.other_stake_sum) }}
              </p>
            </template>
            <template slot="stake_total" slot-scope="data">
              <p class="text-right mb-0" v-if="data.item.stake_total > 0 ">
                {{ formatAmount(data.item.stake_total) }}
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
            aria-controls="candidates-table"
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
      title: 'PolkaStats - Polkadot Kusama phragmen candidates',
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama phragmen candidates' }
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
        { key: 'pub_key_stash', label: 'Candidate', sortable: true, filterByFormatted: true },
        { key: 'other_stake_count', label: 'Voters', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'stake_validator', label: 'Self stake', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'other_stake_sum', label: 'Voters stake', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'stake_total', label: 'Total stake', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` },
        { key: 'favorite', label: '⭐', sortable: true, class: `d-none d-sm-none d-md-table-cell d-lg-table-cell d-xl-table-cell` }
      ],
      blockExplorer,
      favorites: [],
      polling: null
    }
  },
  computed: {
    network() {
      return this.$store.state.network.info;
    },
    candidates() {
      let candidates = [];
      for(let i = 0; i < this.$store.state.phragmen.info.candidates.length; i++) {
        let candidate = this.$store.state.phragmen.info.candidates[i];
        let identity = "";
        if (this.hasIdentity(candidate.pub_key_stash)) {
          identity = this.getIdentity(candidate.pub_key_stash);
        }
        let kusamaIdentity = "";
        if (this.hasKusamaIdentity(candidate.pub_key_stash)) {
          kusamaIdentity = this.getKusamaIdentity(candidate.pub_key_stash);
        }
        candidates.push({
          ...candidate,
          identity,
          kusamaIdentity,
          accountIndex: this.indexes[candidate.pub_key_stash],
          favorite: this.isFavorite(candidate.accountIndex)
        });
      }
      console.log('candidates', candidates)
      return candidates;
    },
    validator_count() {
      return this.$store.state.phragmen.info.validator_count;
    },
    nominator_count() {
      return this.$store.state.phragmen.info.nominator_count;
    },
    total_issuance() {
      return this.$store.state.phragmen.info.total_issuance;
    },
    identities() {
      return this.$store.state.identities.list;
    },
    indexes() {
      return this.$store.state.indexes.list;
    },
    sortOptions() {
      // Create an options list from our fields
      return this.fields
        .filter(f => f.sortable)
        .map(f => {
          return { text: f.label, value: f.key }
        });
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
    
    // Force update of phragmen candidates list if empty
    if (this.$store.state.phragmen.info.candidates.length == 0) {
      vm.$store.dispatch('phragmen/update');
    }
    this.totalRows = this.$store.state.phragmen.info.candidates.length;

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

    // Update data every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('network/update');
      vm.$store.dispatch('phragmen/update');
      vm.$store.dispatch('identities/update');
      vm.$store.dispatch('stakingIdentities/update');
      if (!this.filter) this.totalRows = this.$store.state.phragmen.info.candidates.length;
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
        this.favorites.push({ accountId: validator, name: 'Edit phragmen name...'});
      }
      return true;
    },
    isFavorite(validator) {
      // console.log('isFavoite', this.favorites)
      // Receives validator accountId
      // if (this.favorites != undefined) {
        for (var i=0; i < this.favorites.length; i++) {
          if (this.favorites[i].accountId == validator) {
            return true;
          }
        }
      // }
      // console.log('sale false isFavorite')
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
.favorite {
  cursor: pointer;
  font-size: 1.1rem;
}
.page-phragmen .favorite {
  position: absolute;
/*   top: 0.2rem;
  right: 0.2rem; */
  z-index: 10;
  font-size: 1.1rem;
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
.candidates {
  color: #670d35;
}
.candidate {
  font-size: 0.9rem;
}
.candidate .value {
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
.candidate .col-md-9 .identicon {
  display: inline;
  margin-right: 0.2rem;
  cursor: copy;
}
.candidate .col-md-9 .identicon div {
  display: inline;
}
.identity {
  max-width: 80px;
}
</style>