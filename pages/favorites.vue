<template>
  <div>
    <section>
      <b-container class="page-favorites main pt-4">
        <!-- Empty favorites message -->
        <div class="alert alert-warning alert-dismissible fade show mt-3 mb-4" role="alert" v-if="favorites.length === 0">
          <strong>Hi there!</strong> You can click in the star icon <i class="fas fa-star"></i> of a validator or intention to track it on this page.
          <button type="button" class="close" data-dismiss="alert" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <!-- START FAVORITE VALIDATORS -->
        <template  v-for="(validator, index) in validators">
          <template v-if="isFavorite(validator.accountId)">
            <div class="validator card mb-3">
              <i v-if="validator.imOnline.isOnline" class="imOnline fas fa-check-circle" v-b-tooltip.hover v-bind:title="getImOnlineMessage(validator)"></i>
              <i v-else class="imOffline fas fa-times-circle" v-b-tooltip.hover v-bind:title="getImOnlineMessage(validator)"></i>
              <i v-if="validator.currentElected" class="elected fas fa-chevron-circle-right" v-b-tooltip.hover title="Elected for next session"></i>
              <i v-else class="notElected fas fa-times-circle" v-b-tooltip.hover title="Not elected for next session"></i>
              <a class="favorite" v-on:click="toggleFavorite(validator.accountId)">
                <i v-if="isFavorite(validator.accountId)" class="fas fa-star" style="color: #f1bd23" v-b-tooltip.hover title="Unset as Favorite"></i>
                <i v-else class="fas fa-star" style="color: #e6dfdf;" v-b-tooltip.hover title="Set as Favorite"></i>
              </a>                 
              <div class="card-body">
                <div class="row">
                  <div class="col-md-3 mb-2 text-center">
                    <div v-if="hasIdentity(validator.accountId)">
                      <div v-if="getIdentity(validator.accountId).logo !== ''">
                        <img v-bind:src="getIdentity(validator.accountId).logo" class="identity mt-2" />
                        <h4 class="mt-2 mb-2" v-if="getIdentity(validator.accountId).full_name !== ''">{{ getIdentity(validator.accountId).full_name }}</h4>
                      </div>
                      <div v-else>
                        <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                      </div>
                    </div>
                    <div v-else-if="hasKusamaIdentity(validator.accountId)">
                      <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                      {{ getKusamaIdentity(validator.accountId).display }}
                    </div>
                    <div v-else>
                      <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                    </div>
                    <p class="mt-3 mb-0 rank">
                      rank #{{ index+1 }} <i class="fas fa-shield-alt" v-b-tooltip.hover title="Active validator"></i>
                    </p>
                    <p v-if="validator.stakers.total > 0" class="bonded mb-0" v-b-tooltip.hover title="Total bonded">{{ formatAmount(validator.stakers.total) }}</p>
                    <p v-else class="bonded mb-0" v-b-tooltip.hover title="Total bonded">{{ formatAmount(validator.stakingLedger.total) }}</p>
                    <p class="mb-0">
                      <small>
                        <span v-b-tooltip.hover title="Self bonded" v-if="validator.stakers.own > 0">{{ formatAmount(validator.stakers.own) }}</span>
                        <span v-b-tooltip.hover title="Bonded by nominators" v-if="(validator.stakers.total - validator.stakers.own) > 0">(+{{ formatAmount(validator.stakers.total - validator.stakers.own) }})</span>
                      </small>
                    </p>
                    <p class="mb-0" v-b-tooltip.hover title="Percentage over total bonded stake">{{ getStakePercent(validator.stakers.total, totalStakeBonded) }}% of total stake</p>
                    <p class="mb-0" v-if="validator.currentEraPointsEarned">{{ validator.currentEraPointsEarned }} era points</p>
                  </div>
                  <div class="col-md-9">
                    <h4 class="card-title mb-4 account mt-4 mt-sm-0 mt-md-0 mt-lg-0 mt-xl-0">
                      <nuxt-link :to="{name: 'validator', query: { accountId: validator.accountId } }" title="Validator details">
                        <span v-if="hasIdentity(validator.accountId)">
                          {{ getIdentity(validator.accountId).full_name }}
                        </span>
                        <span v-else-if="hasKusamaIdentity(validator.accountId)">
                          {{ getKusamaIdentity(validator.accountId).display }}
                        </span>
                        <span v-else>
                          {{ validator.accountId }}
                        </span>
                      </nuxt-link>
                    </h4>
                    <div class="row" v-if="validator.stashId">
                      <div class="col-md-3 mb-1">
                        <strong>Stash</strong>
                      </div>
                      <div class="col-md-9 mb-1">
                        <Identicon :value="validator.stashId" :size="20" :theme="'polkadot'" :key="validator.stashId" />
                        <a v-bind:href="blockExplorer.account + validator.stashId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.stashId">{{ indexes[validator.stashId] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.stashId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="validator.controllerId">
                      <div class="col-md-3 mb-1">
                        <strong>Controller</strong>
                      </div>
                      <div class="col-md-9 mb-1">
                        <Identicon :value="validator.controllerId" :size="20" :theme="'polkadot'" :key="validator.controllerId" />
                        <a v-bind:href="blockExplorer.account + validator.controllerId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.controllerId">{{ indexes[validator.controllerId] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.controllerId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="validator.sessionIdHex">
                      <div class="col-md-3 mb-1">
                        <strong>Session id</strong>
                      </div>
                      <div v-bind:id="'val-session-id-info-' + index" class="col-md-9 mb-1">
                        <a class="" data-toggle="collapse" v-bind:href="'#val-session-id-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'val-session-id-' + index">
                          <i class="fas"></i> {{ shortSessionId(validator.sessionIdHex) }}
                        </a>
                        <div class="collapse pt-2 pb-3"  v-bind:id="'val-session-id-' + index" v-bind:data-parent="'#val-session-id-info-' + index">
                          {{ validator.sessionIdHex }}
                        </div>
                      </div>
                    </div>
                    <div class="row" v-if="validator.nextSessionIdHex">
                      <div class="col-md-3 mb-1">
                        <strong>Next session id</strong>
                      </div>
                      <div v-bind:id="'val-next-session-id-info-' + index" class="col-md-9 mb-1">
                        <a class="" data-toggle="collapse" v-bind:href="'#val-next-session-id-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'val-next-session-id-' + index">
                          <i class="fas"></i> {{ shortSessionId(validator.nextSessionIdHex) }}
                        </a>
                        <div class="collapse pt-2 pb-3"  v-bind:id="'val-next-session-id-' + index" v-bind:data-parent="'#val-next-session-id-info-' + index">
                          {{ validator.nextSessionIdHex }}
                        </div>
                      </div>
                    </div>
                    <div class="row" v-if="validator.validatorPrefs.commission">
                      <div class="col-md-3 mb-1">
                        <strong>Commission</strong>
                      </div>
                      <div class="col-md-9 mb-1 fee">
                        {{ (validator.validatorPrefs.commission / 10000000).toFixed(2) }}%
                      </div>
                    </div>
                    <div class="row mb-2">
                      <div class="col-md-3 mb-1">
                        <strong>Reward destination</strong>
                      </div>
                      <div class="col-md-9 mb-1 fee">
                        {{ formatRewardDest(validator.rewardDestination) }}
                      </div>
                    </div>
                    <div v-bind:id="'validator-info-' + index">
                      <template v-if="validator.stakers">
                        <template v-if="validator.stakers.others.length > 0">
                          <a class="" data-toggle="collapse" v-bind:href="'#staker' + index" role="button" aria-expanded="false" v-bind:aria-controls="'staker' + index">
                            <h6 class="h6 nominators d-inline mr-4"><i class="fas"></i> Stakers ({{ validator.stakers.others.length }})</h6>
                          </a>
                        </template>
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
                              <Identicon :value="sessionId" :size="20" :theme="'polkadot'" />                      
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
                      <template v-if="validator.stakers">
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
                      </template>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </template>
        </template>
        <!-- END FAVORITE VALIDATORS -->
        <!-- START FAVORITE INTENTIONS -->
        <template  v-for="(validator, index) in intentions">
          <template v-if="isFavorite(validator.accountId)">
            <div class="validator card mb-3">
              <a class="favorite" v-on:click="toggleFavorite(validator.accountId)">
                <i v-if="isFavorite(validator.accountId)" class="fas fa-star" style="color: #f1bd23" v-b-tooltip.hover title="Unset as Favorite"></i>
                <i v-else class="fas fa-star" style="color: #e6dfdf;" v-b-tooltip.hover title="Set as Favorite"></i>
              </a>                 
              <div class="card-body">
                <div class="row">
                  <div class="col-md-3 mb-2 text-center">
                    <div v-if="hasIdentity(validator.stashId)">
                      <div v-if="getIdentity(validator.stashId).logo !== ''">
                        <img v-bind:src="getIdentity(validator.stashId).logo" class="identity mt-2" />
                        <h4 class="mt-2 mb-2" v-if="getIdentity(validator.stashId).full_name !== ''">{{ getIdentity(validator.stashId).full_name }}</h4>
                      </div>
                      <div v-else>
                        <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                      </div>
                    </div>
                    <div v-else-if="hasKusamaIdentity(validator.stashId)">
                      <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                      {{ getKusamaIdentity(validator.accountId).display }}
                    </div>
                    <div v-else>
                      <Identicon :value="validator.accountId" :size="80" :theme="'polkadot'" :key="validator.accountId" />
                    </div>
                    <p class="mt-3 mb-0 rank">
                      rank #{{ index+1 }} <i class="fas fa-exclamation-triangle" v-b-tooltip.hover title="Intention"></i>
                    </p>
                    <template v-if="validator.stakingLedger">
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
                    </template>
                  </div>
                  <div class="col-md-9">
                    <h4 class="card-title mb-4 account mt-4 mt-sm-0 mt-md-0 mt-lg-0 mt-xl-0">
                      <nuxt-link :to="{name: 'intention', query: { accountId: validator.accountId } }" title="Validator details">
                        <span v-if="hasIdentity(validator.accountId)">
                          {{ getIdentity(validator.accountId).full_name }}
                        </span>
                        <span v-else-if="hasKusamaIdentity(validator.accountId)">
                          {{ getKusamaIdentity(validator.accountId).display }}
                        </span>
                        <span v-else>
                          {{ validator.accountId }}
                        </span>
                      </nuxt-link>
                    </h4>
                    <div class="row" v-if="validator.stashId">
                      <div class="col-md-3 mb-1">
                        <strong>Stash</strong>
                      </div>
                      <div class="col-md-9 mb-1">
                        <Identicon :value="validator.stashId" :size="20" :theme="'polkadot'" :key="validator.stashId" />
                        <a v-bind:href="blockExplorer.account + validator.stashId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.stashId">{{ indexes[validator.stashId] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.stashId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="validator.controllerId">
                      <div class="col-md-3 mb-1">
                        <strong>Controller</strong>
                      </div>
                      <div class="col-md-9 mb-1">
                        <Identicon :value="validator.controllerId" :size="20" :theme="'polkadot'" :key="validator.controllerId" />
                        <a v-bind:href="blockExplorer.account + validator.controllerId" target="_blank">
                          <span class="d-inline d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="validator.controllerId">{{ indexes[validator.controllerId] }}</span>
                          <span class="d-none d-sm-inline d-md-inline d-lg-inline d-xl-inline">{{ indexes[validator.controllerId] }}</span>
                        </a>
                      </div>
                    </div>
                    <div class="row" v-if="validator.sessionIdHex">
                      <div class="col-md-3 mb-1">
                        <strong>Session id</strong>
                      </div>
                      <div v-bind:id="'int-session-id-info-' + index" class="col-md-9 mb-1">
                        <a class="" data-toggle="collapse" v-bind:href="'#int-session-id-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'int-session-id-' + index">
                          <i class="fas"></i> {{ shortSessionId(validator.sessionIdHex) }}
                        </a>
                        <div class="collapse pt-2 pb-3"  v-bind:id="'int-session-id-' + index" v-bind:data-parent="'#int-session-id-info-' + index">
                          {{ validator.sessionIdHex }}
                        </div>
                      </div>
                    </div>
                    <div class="row" v-if="validator.nextSessionIdHex">
                      <div class="col-md-3 mb-1">
                        <strong>Next session id</strong>
                      </div>
                      <div v-bind:id="'int-next-session-id-info-' + index" class="col-md-9 mb-1">
                        <a class="" data-toggle="collapse" v-bind:href="'#int-next-session-id-' + index" role="button" aria-expanded="false" v-bind:aria-controls="'int-next-session-id-' + index">
                          <i class="fas"></i> {{ shortSessionId(validator.nextSessionIdHex) }}
                        </a>
                        <div class="collapse pt-2 pb-3"  v-bind:id="'int-next-session-id-' + index" v-bind:data-parent="'#int-next-session-id-info-' + index">
                          {{ validator.nextSessionIdHex }}
                        </div>
                      </div>
                    </div>
                    <div class="row" v-if="validator.validatorPrefs.commission">
                      <div class="col-md-3 mb-1">
                        <strong>Commission</strong>
                      </div>
                      <div class="col-md-9 mb-1 fee">
                        {{ (validator.validatorPrefs.commission / 10000000).toFixed(2) }}%
                      </div>
                    </div>
                    <div class="row mb-2">
                      <div class="col-md-3 mb-1">
                        <strong>Reward destination</strong>
                      </div>
                      <div class="col-md-9 mb-1 fee">
                        {{ formatRewardDest(validator.rewardDestination) }}
                      </div>
                    </div>
                    <div v-bind:id="'validator-info-' + index">
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
                                <span class="d-inline-block d-sm-none d-md-none d-lg-none d-xl-none" v-b-tooltip.hover v-bind:title="staker.who">{{ shortAddress(staker.who) }}</span>
                                <span class="d-none d-sm-inline-block d-md-inline-block d-lg-inline-block d-xl-inline-block">{{ staker.who }}</span>                        
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
        <!-- END FAVORITE INTENTIONS -->
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
      title: 'PolkaStats - Polkadot Kusama favorite validators and intentions',
      meta: [
        { hid: 'description', name: 'description', content: 'Polkadot Kusama favorite validators and intentions' }
      ]
    }
  },
  mixins: [commonMixin],
  data: function() {
    return {
      blockExplorer,
      favorites: [],
      polling: null
    }
  },
  computed: {
    network () {
      return this.$store.state.network.info;
    },
    validators () {
      return this.$store.state.validators.list
    },
    totalStakeBonded () {
      return this.$store.state.validators.totalStakeBonded
    },
    intentions () {
      return this.$store.state.intentions.list
    },
    identities() {
      return this.$store.state.identities.list
    },
    indexes() {
      return this.$store.state.indexes.list
    },
    totalStakeBondedPercen() {
      if (this.totalStakeBonded !== 0 && this.network.totalIssuance !== "") {
        let totalIssuance = new BN(this.network.totalIssuance, 10);
        let totalStakeBonded = this.totalStakeBonded.mul(new BN('100', 10));
        return totalStakeBonded.div(totalIssuance);
      } else {
        return 0;
      }
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
    
    // Force update of validators list if empty
    if (this.$store.state.validators.list.length == 0) {
      vm.$store.dispatch('validators/update');
    }

    // Force update of indentity list if empty
    if (this.$store.state.identities.list.length == 0) {
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

    // Force update of intention validators list if empty
    if (this.$store.state.intentions.list.length == 0) {
      vm.$store.dispatch('intentions/update');
    }

    // Update network info, validators and intention validators every 10 seconds
    this.polling = setInterval(() => {
      vm.$store.dispatch('network/update');
      vm.$store.dispatch('validators/update');
      vm.$store.dispatch('intentions/update');
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
.page-favorites .favorite {
  position: absolute;
  top: 0.2rem;
  right: 0.2rem;
  z-index: 10;
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
.validator .col-md-9 .identicon {
  display: inline;
  margin-right: 0.2rem;
  cursor: copy;
}
.validator .col-md-9 .identicon div {
  display: inline;
}
.identity {
  max-width: 80px;
}
.validator .fa-shield-alt {
  color: green;
}
.validator .fa-exclamation-triangle {
  color: red;
}
.page-favorites .imOnline {
  position: absolute;
  top: 0.4rem;
  left: 0.4rem;
  font-size: 1.1rem;
  color: green;
}
.page-favorites .imOffline {
  position: absolute;
  top: 0.4rem;
  left: 0.4rem;
  font-size: 1.1rem;
  color: red;
}
.page-favorites .elected {
  position: absolute;
  top: 0.4rem;
  left: 2rem;
  font-size: 1.1rem;
  color: #2697e2;
}
.page-favorites .notElected {
  position: absolute;
  top: 0.4rem;
  left: 2rem;
  font-size: 1.1rem;
  color: red;
}
</style>