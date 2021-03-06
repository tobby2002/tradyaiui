<template>
  <div>
    <div class="q-mb-md">
      <div class="text-h4 q-mb-md">Trade</div>
      <div class="text-subtitle">Run your strategy against a live market!</div>
    </div>
    <div class="row justify-end q-pa-md">
      <!--      <div class="text-h4 q-mb-sm">Start a new live Gekko</div>-->
      <q-btn outline color="teal" @click.prevent="$router.push('/live-gekkos/new')">Start Trade</q-btn>
      <p></p>
    </div>
    <q-space />
    <div class="q-mb-md">
      <div class="text-h6">Market watchers</div>
      <q-banner rounded class="bg-blue-grey-3 text-white" v-if="!watchers.length">
        <template v-slot:avatar>
          <q-icon name="info" color="white"/>
        </template>
        You are currently not watching any markets.
      </q-banner>
      <q-table
        v-if="watchers.length"
        :columns="watchColumns"
        row-key="id"
        :data="watchers"
        :pagination="{rowsPerPage: 0}"
        color="primary"
        separator="horizontal"
        hide-bottom
      >
        <q-tr slot="body" slot-scope="props" :props="props">
          <q-td key="exchange" :props="props">
            {{props.row.config.watch.exchange}}
          </q-td>
          <q-td key="pair" :props="props">
            <!--{{props.row.config.watch.currency}} - {{props.row.config.watch.asset}}-->
            <q-chip class="q-mb-xs"
                    :avatar="'statics/crypto_icons/color/' + props.row.config.watch.currency.toLowerCase() + '.svg'"
                    square dense color="transparent" text-color="black"
            >{{props.row.config.watch.currency}}
            </q-chip>
            -
            <q-chip class="q-mb-xs"
                    :avatar="'statics/crypto_icons/color/' + props.row.config.watch.asset.toLowerCase() + '.svg'"
                    square dense color="transparent" text-color="black"
            >{{props.row.config.watch.asset}}
            </q-chip>
          </q-td>
          <q-td key="startedat" :props="props">
            {{props.row.start ? props.row.start : '' | formatDate}}
          </q-td>
          <q-td key="lastupdate" :props="props">
            {{props.row.latestUpdate ? props.row.latestUpdate : '' | formatDate}}
          </q-td>
          <q-td key="duration" :props="props">
            {{props.row.start && props.row.latestUpdate ? timespan(props.row.start,
            props.row.latestUpdate) : ''}}
          </q-td>
          <q-td key="price" :props="props">
            {{props.row.events.latest.candle ? props.row.events.latest.candle.close + ' ' +
            props.row.config.watch.currency : ''}}
          </q-td>
          <q-td key="actions" :props="props">
            <q-btn outline size="sm" color="secondary" @click="$router.push(`live-gekkos/${props.row.id}`)"
                   icon="visibility" label="view"></q-btn>
            <!--<q-btn outline  size="sm" color="negative" icon="stop" label="stop" @click="stopGekko(props.row.id)"></q-btn>-->
          </q-td>
        </q-tr>
      </q-table>
    </div>
    <div class="q-mb-md">
      <div class="text-h6">Strategy Runners</div>
      <q-banner rounded class="bg-blue-grey-3 text-white" v-if="!stratrunners.length">
        <template v-slot:avatar>
          <q-icon name="info" color="white"/>
        </template>
        You are currently not running any strategies.
      </q-banner>
      <q-table
        v-if="stratrunners.length"
        :columns="stratColumns"
        row-key="id"
        :data="stratrunners"
        :pagination="{rowsPerPage: 0}"
        color="primary"
        separator="horizontal"
        hide-bottom
      >
        <q-tr slot="body" slot-scope="props" :props="props"
              :class="{'bg-green-11': (props.row.events.latest.performanceReport && props.row.events.latest.performanceReport.profit > 0), 'bg-red-11': (props.row.events.latest.performanceReport && props.row.events.latest.performanceReport.profit < 0)}">
          <q-td key="type" :props="props">
            {{props.row.logType}}
          </q-td>
          <q-td key="exchange" :props="props" class="text-uppercase">
            {{props.row.config.watch.exchange}}
          </q-td>
          <q-td key="pair" :props="props">
            <q-chip class="q-mb-xs"
                    :avatar="'statics/crypto_icons/color/' + props.row.config.watch.currency.toLowerCase() + '.svg'"
                    square dense color="transparent" text-color="black"
            >{{props.row.config.watch.currency}}
            </q-chip>
            -
            <q-chip class="q-mb-xs"
                    :avatar="'statics/crypto_icons/color/' + props.row.config.watch.asset.toLowerCase() + '.svg'"
                    square dense color="transparent" text-color="black"
            >{{props.row.config.watch.asset}}
            </q-chip>
          </q-td>
          <q-td key="status" :props="props">
            <q-chip square dense icon="stop" v-if="props.row.stopped" text-color="white" color="orange">Stopped</q-chip>
            <q-chip square dense icon="error" v-if="props.row.errored" text-color="white" color="negative">Error</q-chip>
            <q-chip square dense icon="play_arrow" v-if="props.row.active" text-color="white" color="positive">Running</q-chip>
          </q-td>
          <q-td key="lastupdate" :props="props">
            {{props.row.events.latest.candle ? props.row.events.latest.candle.start : '' | formatDate}}
          </q-td>
          <q-td key="duration" :props="props">
            {{props.row.events.initial.candle && props.row.events.latest.candle ?
            timespan(props.row.events.latest.candle.start,
            props.row.events.initial.candle.start) : ''}}
          </q-td>
          <q-td key="strategy" :props="props">
            {{props.row.config.tradingAdvisor ? props.row.config.tradingAdvisor.method : ''}}
          </q-td>
          <q-td key="trades_rt" :props="props">
            {{(props.row.events.tradeCompleted ? props.row.events.tradeCompleted.length : 0) + ' / ' +
            (props.row.events.roundtrip ? props.row.events.roundtrip.length : 0)}}
          </q-td>
          <q-td key="success" :props="props">
            {{props.row.events.roundtrip ? successRate(props.row.events.roundtrip) : '0.00 %'}}
          </q-td>
          <q-td
            key="profit" :props="props">{{props.row.events.latest.performanceReport ?
            round(props.row.events.latest.performanceReport.profit) : 'N/A' }}
            {{ props.row.events.latest.performanceReport ? props.row.config.watch.currency : ''}}
          </q-td>
          <q-td class="bg-white" key="actions" :props="props">
            <q-btn outline size="sm" color="secondary" @click="$router.push(`live-gekkos/${props.row.id}`)"
                   icon="visibility" label="view"/>
<!--            <q-btn outline  size="sm" color="negative" icon="stop" label="stop" @click="stopGekko(props.row.id)"/>-->
          </q-td>
        </q-tr>
      </q-table>
    </div>
    <div>
<!--      <div class="text-h4 q-mb-sm">Start a new live Gekko</div>-->
<!--      <q-btn outline  color="amber-8" @click.prevent="$router.push('/live-gekkos/new')">START</q-btn>-->
    </div>
  </div>
</template>

<script>
  import moment from "moment";
  import humanizeDuration from "humanize-duration";
  import DateFilters from '../mixins/DateFilterMixin'

  export default {
    mixins: [DateFilters],
    data: () => {
      return {
        timer: false,
        now: moment(),
        watchColumns: [
          {name: "exchange", label: "Exchange"},
          {name: "pair", label: "Pair"},
          {name: "startedat", label: "Started at"},
          {name: "lastupdate", label: "Last update"},
          {name: "duration", label: "running since"},
          {name: "price", label: "Price"},
          {name: "actions", label: "Actions"}
        ],
        stratColumns: [
          {name: "type", label: "Type"},
          {name: "exchange", label: "Exchange"},
          {name: "pair", label: "Pair"},
          {name: "status", label: "Status"},
          {name: "lastupdate", label: "Last update"},
          {name: "duration", label: "Duration"},
          {name: "strategy", label: "Strategy"},
          {name: "trades_rt", label: "Trades/RT"},
          {name: "success", label: "Successful"},
          {name: "profit", label: "Profit"},
          {name: "actions", label: "Actions"}
        ]
      };
    },
    created: function () {
      this.timer = setInterval(() => {
        this.now = moment();
      }, 1000);
    },
    destroyed: function () {
      clearTimeout(this.timer);
    },
    computed: {
      stratrunners: function () {
        return _.values(this.$store.getters['gekkos/list'])
          .concat(_.values(this.$store.getters['gekkos/archive']))
          .filter(g => {
            switch (g.logType) {
              case 'papertrader':
                return true;
              case 'tradebot':
                return true;
              default:
                return false;
            }
          })
      },
      watchers: function () {
        return _.values(this.$store.getters['gekkos/list'])
          .concat(_.values(this.$store.getters['gekkos/archive']))
          .filter(g => g.logType === 'watcher')
      }
    },
    methods: {
      stopGekko(gekkoId) {
        if (!gekkoId) return;
        // find dependant gekkos (important for watchers that have one or more traders)
        let runnerIdx = _.findIndex(this.stratrunners, function (o) {
          return o.id == gekkoId
        });
        let watcherIdx = -1;
        let runnerWatch = {};
        if (runnerIdx >= 0) {
          // the gekko to kill is a stratrunner... so get its watch properties to find the corresponding watcher
          runnerWatch = this.stratrunners[runnerIdx].watch;
          // and look for other stratrunners on the same market
          let otherRunners = _.filter(this.stratrunners, function (o) {
            let l = runnerWatch.asset + runnerWatch.currency + runnerWatch.exchange;
            let r = o.watch.asset + o.watch.currency + o.watch.exchange
            return l == r;
          });
          // if there are multiple runners for one watcher - we do not kill the watcher!
          if (otherRunners.length > 1) {
            // kill single runner only after confirmation...
            this.$q.dialog({
              title: 'Stop Live Gekko',
              color: "warning",
              message: `Do you really want to stop the Stratrunner for [${runnerWatch.exchange} ${runnerWatch.currency}-${runnerWatch.asset}] ?`,
              ok: 'Ok',
              cancel: 'Cancel'
            }).then(() => {
              let wList = gekkoId;
              this.$axios
                .post(this.$store.state.config.apiBaseUrl + "killGekko", {id: wList})
                .then(response => {
                  this.$q.notify({type: 'positive', message: 'Successfully stopped Gekko (ID ' + gekkoId + ')'});
                  this.$store.dispatch('stratrunners/removeRunner', gekkoId);
                })
                .catch(error => {
                  this.$q.notify({
                    type: 'negative',
                    message: 'Error while stopping Gekko (ID ' + gekkoId + ')',
                    detail: error
                  });
                });
            })
          } else {
            // otherwise we ask the user, if he wants to kill the watcher also
            this.$q.dialog({
              title: 'Stop Live Gekko and market watcher',
              color: "info",
              message: `The selected stratrunner
              for [${runnerWatch.exchange} ${runnerWatch.currency}-${runnerWatch.asset}]
              has a dependant market-watcher. What do you want to do?`,
              options: {
                type: 'radio',
                model: '1',
                items: [
                  {label: 'Stop only strategy-runner', value: '1', color: 'primary'},
                  {label: 'Stop strategy-runner AND market-watcher', value: '2', color: 'secondary'}
                ]
              },
              ok: 'Ok',
              cancel: 'Cancel'
            }).then((data) => {
              // if yes, kill stratrunner, and eventually watcher.
              let wList = [];
              wList.push(gekkoId);
              if (data == '2') {
                let wIdx = _.findIndex(this.watchers, function (o) {
                  return o.watch == runnerWatch
                });
                if (wIdx >= 0) wList.push(this.watchers[wIdx].id)
              }
              let self = this;
              wList.forEach(function (id) {
                self.$axios
                  .post(self.$store.state.config.apiBaseUrl + "killGekko", {id: id})
                  .then(response => {
                    self.$q.notify({type: 'positive', message: 'Successfully stopped Gekko (ID ' + id + ')'});
                    self.$store.dispatch('stratrunners/removeRunner', id);
                    self.$store.dispatch('watchers/removeWatcher', id);
                  })
                  .catch(error => {
                    self.$q.notify({
                      type: 'negative',
                      message: 'Error while stopping Gekko (ID ' + id + ')',
                      detail: error
                    });
                  });
              })
            })

          }
        } else {
          // we have a watcher...
          // so find it inside the watchers collection
          watcherIdx = _.findIndex(this.watchers, function (o) {
            return o.id == gekkoId
          });
          if (watcherIdx >= 0) {
            let runnerWatch = this.watchers[watcherIdx].watch;
            // look for stratrunners, that are using that watcher
            let otherRunners = _.filter(this.stratrunners, function (o) {
              let l = runnerWatch.asset + runnerWatch.currency + runnerWatch.exchange;
              let r = o.watch.asset + o.watch.currency + o.watch.exchange;
              return l == r;
            });
            if (otherRunners.length > 0) {
              // if there is one or more, ask if we should kill the whole bunch
              this.$q.dialog({
                title: 'This market watcher has dependant runners!',
                color: "warning",
                message: `The market watcher for [${runnerWatch.exchange} ${runnerWatch.currency}-${runnerWatch.asset}]
                has ${otherRunners.length} dependant strategy-runner(s).
                Do you want to close ALL of them?`,
                ok: 'Yes, close all!',
                cancel: 'No, do nothing!'
              }).then(() => {
                let wList = [];
                wList.push(gekkoId);
                otherRunners.forEach(function (item) {
                  wList.push(item.id);
                });
                let self = this;
                wList.forEach(function (id) {
                  self.$axios
                    .post(self.$store.state.config.apiBaseUrl + "killGekko", {id: id})
                    .then(response => {
                      self.$q.notify({type: 'positive', message: 'Successfully stopped Gekko (ID ' + id + ')'});
                      self.$store.dispatch('stratrunners/removeRunner', id);
                      self.$store.dispatch('watchers/removeWatcher', id);
                    })
                    .catch(error => {
                      self.$q.notify({
                        type: 'negative',
                        message: 'Error while stopping Gekko (ID ' + id + ')',
                        detail: error
                      });
                    });
                })
              })
            } else {
              // watcher has no dependency , so just confirm it's stop
              this.$q.dialog({
                title: 'Stop market watcher',
                message: `Do you really want to stop the market watcher for [${runnerWatch.exchange} ${runnerWatch.currency}-${runnerWatch.asset}] ?`,
                ok: 'Ok',
                cancel: 'Cancel'
              }).then(() => {
                // if yes, kill all stratrunners, then watcher
                this.$axios
                  .post(this.$store.state.config.apiBaseUrl + "killGekko", {id: gekkoId})
                  .then(response => {
                    this.$q.notify({type: 'positive', message: 'Successfully stopped Gekko (ID ' + gekkoId + ')'});
                    this.$store.dispatch('watchers/removeWatcher', gekkkoId);
                  })
                  .catch(error => {
                    this.$q.notify({
                      type: 'negative',
                      message: 'Error while stopping Gekko (ID ' + gekkoId + ')',
                      detail: error
                    });
                  });
              })
            }
          }
        }
      },
      status: state => {
        if (state.errored)
          return 'errored';
        if (state.stopped)
          return 'stopped';
        if (state.active)
          return 'running';

        console.log('unknown state:', state);
      },
      report: state => {
        return _.get(state, 'events.latest.performanceReport');
      },
      moment: mom => moment.utc(mom),
      round: n => (+n).toFixed(3),
      timespan: function (a, b) {
        return humanizeDuration(this.moment(a).diff(this.moment(b)));
      },
      successRate: rt => {
        if (!rt || !rt.length) return (0).toFixed(2) + " %"

        let successful = rt.filter(function (item) {
          return item.pnl > 0
        })
        return ((+successful.length / rt.length) * 100).toFixed(2) + " %"
      }
    }
  };
</script>
