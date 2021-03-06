<template>
  <span class="network-selector" :title="blockHeight">
    <span class="network-icon tooltipped tooltipped-no-delay tooltipped-n" :class="networkStatusClass" v-tooltip="networkStatusLabel"></span>
    {{chainId}}
    (#{{blockHeight}})
  </span>
</template>

<script>
import { promisifySimple } from '../../utils/promisify';
import { mapState, mapActions } from 'vuex'

let readTimeout = null;

export default {
  data () {
    return {
      status: 'load',
      blockHeight: 0,
      chainId: ''
    }
  },
  created () {
    this.updateStatus();
  },
  beforeDestroy () {
  },
  computed: {
    ...mapState({
      activeChainId: state => state.navigation.activeAccount.chainId
    }),
    networkStatusClass () {
      return `network-${this.status}`;
    },
    networkStatusLabel () {
      return {
        ok: 'connected',
        fail: 'disconnected',
        load: 'connecting...'
      }[this.status];
    }
  },
  watch: {
    'activeChainId': function() {
      this.status = 'load';
      this.updateStatus();
    }
  },
    
  methods: {
    updateStatus () {
      if (readTimeout) clearTimeout(readTimeout);
      promisifySimple(this.$background.getBlockchainStatus)({ chainId: this.activeChainId }).then(status => {
        this.status = 'ok';
        this.blockHeight = status.blockHeight;
        this.chainId = status.chainId;

        readTimeout = setTimeout(() => {
          this.updateStatus();
        }, 5000);
      }).catch(error => {
        this.status = 'fail';
        console.error('Could not connect to blockchain.', error);

        readTimeout = setTimeout(() => {
          this.updateStatus();
        }, 30000); // Retry after 30s
      });
    }
  },
  components: {
  }
};
</script>

<style lang="scss">
.network-selector {
  font-size: 75%;
  text-transform: uppercase;
  font-weight: 500;
}
.network-icon {
  display: inline-block;
  vertical-align: text-bottom;
  width: 9px;
  height: 9px;
  margin-right: 2px;
  transform: translateY(-1px);
  background: url(~@assets/img/connection-load.svg) 50% 50% no-repeat;

  &.network-ok {
    background-image: url(~@assets/img/connection-ok.svg);
  }
  &.network-fail {
    background-image: url(~@assets/img/connection-fail.svg);
  }
}
</style>