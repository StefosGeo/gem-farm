<template>
  <!--control buttons-->
  <div class="mb-20 flex justify-evenly">
    <slot />
  </div>

  <!--wallet + vault view-->
  <div class="flex justify-evenly">
    <!--left-->
    <NFTGrid
      title="Your wallet"
      bg_color="#474747"
      class="flex-1 container"
      :nfts="desiredWalletNFTs"
      @selected="handleWalletSelected"
    />

    <!--mid-->
    <div class="p-16 m-5 flex flex-col justify-center align-start">
      <ArrowButton
        :disabled="vaultLocked"
        class="my-2 pb-20 justify-center align-center"
        @click="moveNFTsFE(false)"
      />
      <ArrowButton
        :disabled="vaultLocked"
        class="my-2 justify-center align-center"
        :left="true"
        @click="moveNFTsFE(true)"
      />
      <div class="pt-12">
        <button
          v-if="
            (toWalletNFTs && toWalletNFTs.length) ||
            (toVaultNFTs && toVaultNFTs.length)
          "
          class="nes-btn is-primary is-warning"
          @click="moveNFTsOnChain"
        >
          Confirm
        </button>
      </div>
    </div>

    <!--right-->
    <NFTGrid
      v-if="bank && vault"
      title="Your vault"
      bg_color="#f07604"
      class="flex-1 vault-container"
      :nfts="desiredVaultNFTs"
      @selected="handleVaultSelected"
    >
      <div
        v-if="vaultLocked"
        class="locked flex-col justify-center items-center align-center"
      >
        <p class="mt-10">This vault is locked!</p>
      </div>
    </NFTGrid>
  </div>
  <!-- <div class="pt-5 flex-wrap justify-center" style="width: 120px">
    <button
      v-if="
        (toWalletNFTs && toWalletNFTs.length) ||
        (toVaultNFTs && toVaultNFTs.length)
      "
      class="nes-btn is-primary mr-8"
      @click="moveNFTsOnChain"
    >
      Confirm moving NFTs
    </button>
  </div> -->
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, watch } from 'vue';
import NFTGrid from '@/components/gem-bank/NFTGrid.vue';
import ArrowButton from '@/components/ArrowButton.vue';
import useWallet from '@/composables/wallet';
import useCluster from '@/composables/cluster';
import {
  getNFTMetadataForMany,
  getNFTsByOwner,
  INFT,
} from '@/common/web3/NFTget';
import { initGemBank } from '@/common/gem-bank';
import { PublicKey } from '@solana/web3.js';
import { getListDiffBasedOnMints, removeManyFromList } from '@/common/util';
import { BN } from '@project-serum/anchor';

export default defineComponent({
  components: { ArrowButton, NFTGrid },
  props: {
    vault: String,
  },
  emits: ['selected-wallet-nft'],
  setup(props, ctx) {
    const { wallet, getWallet } = useWallet();
    const { cluster, getConnection } = useCluster();

    // --------------------------------------- state

    //current walet/vault state
    const currentWalletNFTs = ref<INFT[]>([]);
    const currentVaultNFTs = ref<INFT[]>([]);
    //selected but not yet moved over in FE
    const selectedWalletNFTs = ref<INFT[]>([]);
    const selectedVaultNFTs = ref<INFT[]>([]);
    //moved over in FE but not yet onchain
    const desiredWalletNFTs = ref<INFT[]>([]);
    const desiredVaultNFTs = ref<INFT[]>([]);
    //moved over onchain
    const toWalletNFTs = ref<INFT[]>([]);
    const toVaultNFTs = ref<INFT[]>([]);

    // --------------------------------------- populate initial nfts

    const populateWalletNFTs = async () => {
      // zero out to begin with
      currentWalletNFTs.value = [];
      selectedWalletNFTs.value = [];
      desiredWalletNFTs.value = [];

      if (getWallet()) {
        currentWalletNFTs.value = await getNFTsByOwner(
          getWallet()!.publicKey!,
          getConnection()
        );
        desiredWalletNFTs.value = [...currentWalletNFTs.value];
      }
    };

    const populateVaultNFTs = async () => {
      // zero out to begin with
      currentVaultNFTs.value = [];
      selectedVaultNFTs.value = [];
      desiredVaultNFTs.value = [];

      const foundGDRs = await gb.fetchAllGdrPDAs(vault.value);
      if (foundGDRs && foundGDRs.length) {
        gdrs.value = foundGDRs;
        console.log(`found a total of ${foundGDRs.length} gdrs`);

        const mints = foundGDRs.map((gdr: any) => {
          return { mint: gdr.account.gemMint };
        });
        currentVaultNFTs.value = await getNFTMetadataForMany(
          mints,
          getConnection()
        );
        desiredVaultNFTs.value = [...currentVaultNFTs.value];
        console.log(
          `populated a total of ${currentVaultNFTs.value.length} vault NFTs`
        );
      }
    };

    const updateVaultState = async () => {
      vaultAcc.value = await gb.fetchVaultAcc(vault.value);
      bank.value = vaultAcc.value.bank;
      vaultLocked.value = vaultAcc.value.locked;
    };

    watch([wallet, cluster], async () => {
      gb = await initGemBank(getConnection(), getWallet()!);

      //populate wallet + vault nfts
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    });

    onMounted(async () => {
      gb = await initGemBank(getConnection(), getWallet()!);

      //prep vault + bank variables
      vault.value = new PublicKey(props.vault!);
      await updateVaultState();

      //populate wallet + vault nfts
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    });

    // --------------------------------------- moving nfts

    const handleWalletSelected = (e: any) => {
      if (e.selected) {
        selectedWalletNFTs.value.push(e.nft);
      } else {
        const index = selectedWalletNFTs.value.indexOf(e.nft);
        selectedWalletNFTs.value.splice(index, 1);
      }
      ctx.emit('selected-wallet-nft', selectedWalletNFTs.value);
    };

    const handleVaultSelected = (e: any) => {
      if (e.selected) {
        selectedVaultNFTs.value.push(e.nft);
      } else {
        const index = selectedVaultNFTs.value.indexOf(e.nft);
        selectedVaultNFTs.value.splice(index, 1);
      }
    };

    const moveNFTsFE = (moveLeft: boolean) => {
      if (moveLeft) {
        //push selected vault nfts into desired wallet
        desiredWalletNFTs.value.push(...selectedVaultNFTs.value);
        //remove selected vault nfts from desired vault
        removeManyFromList(selectedVaultNFTs.value, desiredVaultNFTs.value);
        //empty selection list
        selectedVaultNFTs.value = [];
      } else {
        //push selected wallet nfts into desired vault
        desiredVaultNFTs.value.push(...selectedWalletNFTs.value);
        //remove selected wallet nfts from desired wallet
        removeManyFromList(selectedWalletNFTs.value, desiredWalletNFTs.value);
        //empty selected walelt
        selectedWalletNFTs.value = [];
      }
      // console.log("invoce on ChainS")
      // moveNFTsOnChain();
    };

    //todo jam into single tx
    const moveNFTsOnChain = async () => {
      console.log('trying to move gems');
      for (const nft of toVaultNFTs.value) {
        console.log(nft);
        const creator = new PublicKey(
          //todo currently simply taking the 1st creator
          (nft.onchainMetadata as any).data.creators[0].address
        );
        console.log('creator is', creator.toBase58());
        await depositGem(nft.mint, creator, nft.pubkey!);
      }
      for (const nft of toWalletNFTs.value) {
        await withdrawGem(nft.mint);
      }
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    };

    //to vault = vault desired - vault current
    watch(
      desiredVaultNFTs,
      () => {
        toVaultNFTs.value = getListDiffBasedOnMints(
          desiredVaultNFTs.value,
          currentVaultNFTs.value
        );
        console.log('to vault nfts are', toVaultNFTs.value);
      },
      { deep: true }
    );

    //to wallet = wallet desired - wallet current
    watch(
      desiredWalletNFTs,
      () => {
        toWalletNFTs.value = getListDiffBasedOnMints(
          desiredWalletNFTs.value,
          currentWalletNFTs.value
        );
        console.log('to wallet nfts are', toWalletNFTs.value);
      },
      { deep: true }
    );

    // --------------------------------------- gem bank

    let gb: any;
    const bank = ref<PublicKey>();
    const vault = ref<PublicKey>();
    const vaultAcc = ref<any>();
    const gdrs = ref<PublicKey[]>([]);
    const vaultLocked = ref<boolean>(false);

    const depositGem = async (
      mint: PublicKey,
      creator: PublicKey,
      source: PublicKey
    ) => {
      const { txSig } = await gb.depositGemWallet(
        bank.value,
        vault.value,
        new BN(1),
        mint,
        source,
        creator
      );
      console.log('deposit done', txSig);
    };

    const withdrawGem = async (mint: PublicKey) => {
      const { txSig } = await gb.withdrawGemWallet(
        bank.value,
        vault.value,
        new BN(1),
        mint
      );
      console.log('withdrawal done', txSig);
    };

    // --------------------------------------- return

    return {
      wallet,
      desiredWalletNFTs,
      desiredVaultNFTs,
      toVaultNFTs,
      toWalletNFTs,
      handleWalletSelected,
      handleVaultSelected,
      moveNFTsFE,
      moveNFTsOnChain,
      bank,
      // eslint-disable-next-line vue/no-dupe-keys
      vault,
      vaultLocked,
    };
  },
});
</script>

<style scoped>
.container {
  background-color: #363636;
  opacity: 0.7;
}
.locked {
  @apply text-center bg-black text-white;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.7;
  z-index: 10;
}
</style>
