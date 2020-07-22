<template>
  <v-app>
    <v-card class="ma-10 pb-10">
      <v-tabs fixed-tabs v-model="tab">
        <v-tab v-for="item in items" :key="item.tab">
          {{ item.tab }}
        </v-tab>
      </v-tabs>

      <v-tabs-items v-model="tab">
        <v-tab-item v-for="item in items" :key="item.tab">
          <div v-if="item.tab == 'Mint'">
            <v-card class="mx-10">
              <v-row class="pa-5 d-flex justify-center">
                <v-col>
                  Enroll Course
                </v-col>
                <v-col cols="12">
                  <v-select
                    :items="options"
                    v-model="course"
                    placeholder="Course"
                  />
                </v-col>
                <v-col cols="12"
                  ><v-text-field
                    v-model="address"
                    type="text"
                    placeholder="wallet address"
                /></v-col>
                <v-btn @click="mint()">Enroll</v-btn>
              </v-row>
            </v-card>
            <div class="pt-5">
                <v-card dark class="pa-5 mx-10 mb-5">
                  <div v-html="notification.mint"></div>
                </v-card>
              </div>
          </div>
          <div v-if="item.tab == 'Search'">
            <v-card class="mx-10">
              <v-row class="pa-5 d-flex justify-center">
                <v-col>
                  Check Transaction
                </v-col>
                <v-col cols="12"
                  ><v-text-field v-model="hash" type="text" placeholder="Hash"
                /></v-col>
                <v-btn @click="search">Check</v-btn>
              </v-row>
            </v-card>
            <div class="pt-5">
                <v-card dark class="pa-5 mx-10">
                <div>Transaction Type: <span style="color:yellow">{{notification.search.type}}</span></div>
                <div>University: <span style="color:yellow">{{notification.search.owner}}</span></div>
                <div>Token: <span style="color:yellow">{{notification.search.token}}</span></div>
                <div>To: <span style="color:yellow">{{notification.search.to}}</span></div>
                <div>Properties: <span style="color:yellow">{{notification.search.properties}}</span></div>
                <div>Metadata: <span style="color:yellow">{{notification.search.metadata}}</span></div>
              </v-card>
              </div>
          </div>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
    <v-overlay opacity="0.9" v-if="loading">
      {{ msg }}
    </v-overlay>
  </v-app>
</template>

<script>
import { mxw } from "mxw-sdk-js";
import { hexlify, randomBytes, bigNumberify } from "mxw-sdk-js/dist/utils";
import {
  NonFungibleTokenActions,
  NonFungibleToken,
} from "mxw-sdk-js/dist/non-fungible-token";

export default {
  name: "App",
  async created() {
    this.loading = true;
    await this.init();
    this.providerConnection.getBlockNumber().then((blockNumber) => {
      console.log("Latest block number: " + blockNumber);
    });
    for (var i = 1; i < 6; i++) {
      await this.create(i);
    }
  },
  watch: {
    course(value) {
      console.log(value);
    },
  },
  data: () => ({
    loading: false,
    msg: "",
    notification: {
      mint: "Hello",
      search: {
        owner: "",
        type: "",
        value: "",
        from: "",
        to: "",
        token: "",
      },
    },
    balance: 0,
    address: "",
    value: "",
    silentRpc: "",
    providerConnection: mxw.providers.Provider,
    provider: mxw.Wallet,
    issuer: mxw.Wallet,
    middleware: mxw.Wallet,
    token: NonFungibleToken,
    tokenList: [],
    tab: null,
    course: "",
    courseName: "",
    hash: "",
    receipt: {},
    options: [],
    items: [
      { tab: "Mint", content: "test" },
      { tab: "Search", content: "test" },
    ],
  }),
  methods: {
    init() {
      this.msg = "Initializing blockchain...";
      let connection = {
        url: "http://localhost:26657",
        timeout: 60000,
      };
      this.providerConnection = new mxw.providers.JsonRpcProvider(connection, {
        chainId: "maxonrow-chain",
        name: "mxw",
      });
      const providerMnemonic =
        "language indoor mushroom gold motor genuine tower ripple baby journey where offer crumble chuckle velvet dizzy trigger owner mother screen panic question cliff dish";
      this.provider = mxw.Wallet.fromMnemonic(providerMnemonic).connect(
        this.providerConnection
      );

      const issuerMnemonic =
        "appear scale write grow tiger puppy trick kite exhibit distance target cliff coin silly because train matrix weather list chat stamp warfare hobby ocean";
      this.issuer = mxw.Wallet.fromMnemonic(issuerMnemonic).connect(
        this.providerConnection
      );

      const middlewareMnemonic =
        "guard loop tell accuse village list prevent sea dolphin weapon own track spike venue gun blind carry hawk weapon track rain amazing author eagle";
      this.middleware = mxw.Wallet.fromMnemonic(middlewareMnemonic).connect(
        this.providerConnection
      );
    },
    create(i) {
      this.msg = "Generating course " + i + "...";
      let nonFungibleTokenProperties = null;
      let symbol = hexlify(randomBytes(4)).substring(2);
      nonFungibleTokenProperties = {
        name: "NFT" + symbol,
        symbol: symbol,
        fee: {
          to: "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
          value: bigNumberify("1"),
        },
        metadata: "",
        properties: "",
      };
      let overrides = {
        tokenFees: [
          { action: NonFungibleTokenActions.transfer, feeName: "default" },
          {
            action: NonFungibleTokenActions.transferOwnership,
            feeName: "default",
          },
          {
            action: NonFungibleTokenActions.acceptOwnership,
            feeName: "default",
          },
        ],
        endorserList: [],
        mintLimit: -1,
        transferLimit: 1,
        burnable: true,
        transferable: true,
        modifiable: true,
        pub: false,
      };
      return NonFungibleToken.create(
        nonFungibleTokenProperties,
        this.issuer
      ).then((nfToken) => {
        console.log("Token had been created");
        switch (i) {
          case 1:
            this.options.push({ text: "Supernatural Course", value: symbol });
            break;
          case 2:
            this.options.push({ text: "Psychic Course", value: symbol });
            break;
          case 3:
            this.options.push({ text: "Maths Course", value: symbol });
            break;
          case 4:
            this.options.push({ text: "Geography Course", value: symbol });
            break;
          case 5:
            this.options.push({ text: "Computer Course", value: symbol });
            break;
          default:
            this.options.push({ text: "404", value: symbol });
            break;
        }
        // this.options.push(symbol);
        return this.authourize(
          NonFungibleToken.approveNonFungibleToken,
          nfToken.symbol,
          overrides,
          i
        );
      });
    },
    authourize(perform, symbol, overrides, i) {
      this.msg = "Authorizing course " + i + "...";
      return perform(symbol, this.provider, overrides)
        .then((transaction) => {
          return NonFungibleToken.signNonFungibleTokenStatusTransaction(
            transaction,
            this.issuer
          );
        })
        .then((transaction) => {
          return NonFungibleToken.sendNonFungibleTokenStatusTransaction(
            transaction,
            this.middleware
          );
        })
        .then((receipt) => {
          if (receipt.status == 1) console.log("success");
          else console.log("error");
          return this.Query(symbol, i);
        });
    },
    Query(symbol, i) {
      this.msg = "Updating course " + i + "...";
      return NonFungibleToken.fromSymbol(symbol, this.issuer).then(
        (nfToken) => {
          console.log("Updated");
          if (i == 5) this.loading = false;
          return this.tokenList.push(nfToken);
          // return (this.token = nfToken);
        }
      );
    },
    getBalance(token) {
      return token.getBalance().then((balance) => {
        console.log(
          "Total " +
            token.state.name +
            " Token Owned: " +
            mxw.utils.formatUnits(balance, token.state.decimals)
        );
        return balance;
      });
    },
    mint() {
      this.loading = true;
      this.msg = "Minting item...";
      const str = this.course;
      const index = this.options.findIndex((item) => {
        if (str == item.value) return item;
      });
      var nft = this.tokenList[index];
      this.courseName = this.options[index].text;
      const itemProp = {
        symbol: this.course,
        itemID: "course :" + this.course,
        properties: this.courseName + " from University of Reading",
        metadata: "Owned by: " + this.address,
      };
      nft.mint(this.address, itemProp).then((receipt) => {
        this.hash = receipt.hash;
        if (receipt.status == 1)
          this.notification.mint = "You have enrolled <span style='color:green'>" + this.courseName + "</span>";
        else this.notification.mint = "<span style='color:red'>Failed to enroll</span>";

        this.loading = false;
      });
    },
    search() {
      this.providerConnection
        .getTransactionReceipt(this.hash)
        .then((receipt) => {
          this.notification.search.type = receipt.payload.value.msg[0].type;
          this.notification.search.owner = receipt.payload.value.msg[0].value.owner;
          this.notification.search.properties = receipt.payload.value.msg[0].value.properties;
          this.notification.search.metadata = receipt.payload.value.msg[0].value.metadata;
          this.notification.search.to = receipt.payload.value.msg[0].value.to;
          this.notification.search.token = receipt.payload.value.msg[0].value.symbol;
          return (this.receipt = receipt);
        });
    },
  },
};
</script>
