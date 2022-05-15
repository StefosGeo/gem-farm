<template>
  <div class="nes-container flex text-center mb-10 flex-wrap">
    <div class="text-center flex-1 mb-10 width100 select">
      <select
        required
        id="wallet"
        class="huVjiU text-left px-6 choose-wallet"
        v-model="chosenWallet"
      >
        <option class="text-left" disabled hidden :value="null">
          Select Platform
        </option>

        <option :value="WalletName.Phantom">Phantom</option>

        <option :value="WalletName.Sollet">Sollet</option>
        <option :value="WalletName.SolletExtension">Sollet Extension</option>
        <option :value="WalletName.Solflare">Solflare</option>
        <option :value="WalletName.SolflareWeb">Solflare Web</option>
      </select>
    </div>
    <div
      v-if="farmerAcc"
      class="connected-wallet flex justify-center align-middle width100"
    >
      Connected Wallet:
      <p class="flex ml-3">{{ farmerAcc.identity.toBase58() }}...</p>
    </div>
    <!-- <img>there will be an image here to disconnect wallet</img> -->
  </div>
</template>

<script lang="ts">
import { computed, defineComponent } from 'vue';
import { WalletName } from '@solana/wallet-adapter-wallets';
import useCluster, { Cluster } from '@/composables/cluster';
import useWallet from '@/composables/wallet';

export default defineComponent({
  setup() {
    // cluster
    const { cluster, setCluster, getClusterURL } = useCluster();
    setCluster(Cluster.Mainnet);

    const chosenCluster = computed({
      get() {
        return cluster.value;
      },
      set(newVal: Cluster) {
        setCluster(newVal);
      },
    });

    // wallet
    const { getWalletName, setWallet } = useWallet();
    const chosenWallet = computed({
      get() {
        return getWalletName();
      },
      set(newVal: WalletName | null) {
        setWallet(newVal, getClusterURL());
      },
    });

    return {
      Cluster,
      chosenCluster,
      WalletName,
      chosenWallet,
    };
  },
});
</script>

<style>
.select {
  position: relative;
  display: inline-block;
  color: #555;
}

.select select {
  padding: 0.5em 2.25em 0.5em 1em;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-image: url('../assets/angle-down-solid.svg');
  background-repeat: no-repeat;
  background-repeat: no-repeat, repeat;
  background-position: right 0.7em top 50%, 0 0;
}
.option {
  height: 100px;
  background-color: red;
}
select,
:hover {
  cursor: pointer;
}
</style>
