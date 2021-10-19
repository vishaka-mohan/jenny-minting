<script lang="ts">
    import { onMount } from "svelte";
    import { TezosToolkit, MichelCodecPacker, MichelsonMap } from "@taquito/taquito";
    import { char2Bytes, bytes2Char } from "@taquito/utils";
    import { BeaconWallet } from "@taquito/beacon-wallet";
    import { NetworkType } from "@airgap/beacon-sdk";
  
    let Tezos: TezosToolkit;
    let wallet: BeaconWallet;
    const walletOptions = {
      name: "Jenny",
      preferredNetwork: NetworkType.FLORENCENET
    };
    let userAddress: string;
    let token_price, token_id;
    if (process.env.NODE_ENV === "dev") {
      //title = "uranus";
      
    }
  
    const rpcUrl = "https://api.tez.ie/rpc/florencenet";
    const serverUrl =
      process.env.NODE_ENV !== "production"
        ? "http://localhost:8080"
        : "https://my-cool-backend-app.com";
    const contractAddress = "KT1F2BcW3XLPVvHJiRe9X1LswsPYmXCQVUjp";
    let nftStorage = undefined;
    let userNfts: { tokenId: number; ipfsHash: string }[] = [];
    let addingToken = false;
    let newNft:
      | undefined
      | { imageHash: string; metadataHash: string; opHash: string };
  
    const getMarketNFTs = async () => {
      // finds all market NFTs
      const contract = await Tezos.wallet.at(contractAddress);
      nftStorage = await contract.storage();
      console.log(nftStorage);
    };
  
    const connect = async () => {
      if (!wallet) {
        wallet = new BeaconWallet(walletOptions);
      }
  
      try {
        await wallet.requestPermissions({
          network: {
            type: NetworkType.FLORENCENET,
            rpcUrl
          }
        });
        userAddress = await wallet.getPKH();
        Tezos.setWalletProvider(wallet);
        await getMarketNFTs();
      } catch (err) {
        console.error(err);
      }
    };
  
    const disconnect = () => {
      wallet.client.destroy();
      wallet = undefined;
      userAddress = "";
    };
  
    const upload = async () => {
      try {
        
        const data = new FormData();
        data.append("token_id", token_id);
        data.append("token_price", token_price);
          console.log(data)
          addingToken = true;
            // saves NFT on-chain
            const contract = await Tezos.wallet.at(contractAddress);
            const op = await contract.methods
              .mint()
              .send();
            console.log("Op hash:", op.opHash);
            await op.confirmation();
  
  
            
  
            // refreshes storage
            await getMarketNFTs();
         
        
      } catch (error) {
        console.log(error);
      } finally {
        
        addingToken = false;
      }
    };
  
    onMount(async () => {
      Tezos = new TezosToolkit(rpcUrl);
      Tezos.setPackerProvider(new MichelCodecPacker());
      wallet = new BeaconWallet(walletOptions);
      if (await wallet.client.getActiveAccount()) {
        userAddress = await wallet.getPKH();
        Tezos.setWalletProvider(wallet);
        await getMarketNFTs();
      }
    });
  </script>
  
  <style lang="scss">
    $tezos-blue: #2e7df7;
  
    h1 {
      font-size: 3rem;
      font-family: Arial, Helvetica, sans-serif;
    }
  
    button {
      padding: 20px;
      font-size: 1rem;
      border: solid 3px #d1d5db;
      background-color: #ffe6e6;
      border-radius: 10px;
      cursor: pointer;
    }
  
    .roman {
      text-transform: uppercase;
      font-family: Arial, Helvetica, sans-serif;
      font-weight: bold;
    }
  
    .container {
      font-size: 1.3rem;
      & > div {
        padding: 20px;
      }
  
      label {
        display: flex;
        flex-direction: column;
        text-align: left;
      }
  
      input,
      textarea {
        padding: 10px;
      }
  
      .user-nfts {
        display: flex;
        justify-content: center;
        align-items: center;
      }
    }
  </style>
  
  <main>
    <div class="container">
      <h1>MINTING NFTS FOR JENNY</h1>
      {#if userAddress}
        <div>
          <button class="roman" on:click={disconnect}>Disconnect</button>
        </div>
        {#if newNft}
          <div>NFT MINTED</div>
          
          
          <div>
            <a
              href={`https://better-call.dev/edo2net/opg/${newNft.opHash}/contents `}
              target="_blank"
              rel="noopener noreferrer nofollow"
            >
              bcd
            </a>
          </div>
          
        {:else}
          
          <div>
            <label for="image-id">
              <span>Token id:</span>
              <input type="number" id="image-id" bind:value={token_id} />
            </label>
          </div>
          <div>
            <label for="image-name">
              <span>Token price:</span>
              <input type="number" id="image-price" bind:value={token_price} />
            </label>
          </div>
  
  
          
          <div>
            
            {#if addingToken}
              <button class="roman"> Adding to market </button>
            {:else}
              <button class="roman" on:click={upload}> Add </button>
            {/if}
          </div>
        {/if}
      {:else}
        <div class="roman">ADDING CARDS TO MARKET</div>
        <button class="roman" on:click={connect}>Connect your wallet</button>
      {/if}
    </div>
  </main>
  