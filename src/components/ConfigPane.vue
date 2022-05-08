<template>
<div class="nes-container flex text-center mb-10 flex-wrap">
    <div class="nes-select text-center flex-1 mb-10 width100">
      <select required id="wallet" class="huVjiU choose-wallet" v-model="chosenWallet">
        <option class="text-gray-500" :value="null">Choose wallet..</option>
        <option :value="WalletName.Phantom">Phantom</option>
        <option :value="WalletName.Sollet">Sollet</option>
        <option :value="WalletName.SolletExtension">Sollet Extension</option>
        <option :value="WalletName.Solflare">Solflare</option>
        <option :value="WalletName.SolflareWeb">Solflare Web</option>
      </select>
    </div>
    <div v-if="farmerAcc" class="connected-wallet flex justify-center align-middle width100">Connected Wallet: <p class="flex ml-3"> {{farmerAcc.identity.toBase58()}}...</p></div>
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
    setCluster(Cluster.Devnet);

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

<style scoped>

</style>
