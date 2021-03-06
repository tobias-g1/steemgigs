/* eslint-disable indent */
<template>
  <div class="card-action">
    <a v-if="unvoting" v-tooltip="{content: 'please wait'}">
      <i class="fa fa-spinner fa-pulse"></i>
    </a>
    <a v-if="!unvoting" :class="!upvoted ? 'grey-text' : 'indigo-text'" @click="vote" v-tooltip="voteBtnTitle"><i class="fa fa-thumbs-up" aria-hidden="true"></i> {{ upvotes }}</a>
    <a v-tooltip="{ content: 'comment', classes: ['tooltip'] }" ><i class="icon ion-chatbox-working" aria-hidden="true"></i> {{ comments }}</a>
    <a v-if="rsspinning" v-tooltip="{content: 'please wait'}">
      <i class="fa fa-spinner fa-pulse"></i>
    </a>
    <a v-if="resteeming && this.gigData.author !== this.$store.state.username" v-tooltip="{ content: 'resteem', classes: ['tooltip'] }" :class="!resteem ? 'grey-text' : 'indigo-text'" @click="reblog(gigData)"><i class="icon ion-ios-redo" aria-hidden="true"></i></a>
    <span class="right" v-tooltip="{ content: paymentInfo, classes: ['tooltip'] }">${{ payout.toString().slice(0, 4) }}</span>
    <div class="row pt-3 mb-0" v-if="upvoteActive">
      <div class="col s9">
       <slider-range :min="1" v-model="upvoteRange" />
      </div>
      <div class="col offset-s1 s2">
        <a v-if="voting" v-tooltip="{content: 'please wait'}">
          <i class="fa fa-spinner fa-pulse"></i>
        </a>
        <a @click="upvote" v-tooltip="voteBtnTitle" class="upvote-btn" :class="upvoted ? 'upvoted' : ''" v-if="!voting">
          <i class="ion-chevron-up"></i>
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import sc2 from '@/services/sc2'
import SliderRange from 'vue-slider-component'
import moment from 'moment'
import userStatus from '@/mixins/status.js'
import actions from '@/mixins/actions.js'

export default {
  mixins: [userStatus, actions],
  components: {
    SliderRange
  },
  data () {
    return {
      taskDetails: {
        type: String
      },
      voting: false,
      unvoting: false,
      resteeming: true,
      taskPicture: '',
      upvoteActive: false,
      upvoteRange: 100,
      resteem: false,
      rsspinning: false

    }
  },
  props: {
    gigData: Object,
    type: {
      type: String,
      default: 'steemgigs'
    }
  },
  computed: {
    sellerUsername () {
      return this.gigData.author
    },
    comments () {
      return this.gigData.children
    },
    genuineVoters () {
      if (this.gigData.active_votes) {
        return this.gigData.active_votes.filter((x) => x.weight !== 0)
      } else {
        return []
      }
    },
    upvotes () {
      return this.genuineVoters.length
    },
    payout: function () {
      let postCreationDate = moment(this.gigData.created)._d
      let currentDate = moment()._d
      let timesince = moment(currentDate).diff(postCreationDate, 'minutes')
      // If post is greater than 7 days
      if (timesince > 10080) {
        return parseFloat(this.gigData.curator_payout_value) + parseFloat(this.gigData.total_payout_value)
      } else {
        return this.gigData.pending_payout_value
      }
    },
    paymentInfo () {
      if ((new Date(this.gigData.cashout_time).getTime()) > (new Date().getTime())) {
        return `Will payout in ${Math.floor((new Date(this.gigData.cashout_time) - (new Date())) / (1000 * 60 * 60 * 24))} days`
      } else {
        return `Author Payout: ${'$' + this.gigData.total_payout_value}
        Curator Payout: ${'$' + this.gigData.curator_payout_value}`
      }
    },
    myVote () {
      if (this.gigData.active_votes) {
        return this.genuineVoters.filter((x) => x.voter === this.$store.state.username)
      } else {
        return []
      }
    },
    upvoted () {
      return this.myVote.length > 0
    },
    voteBtnTitle () {
      if (this.upvoted) {
        return { content: 'unvote', classes: ['tooltip'] }
      } else {
        return { content: 'upvote', classes: ['tooltip'] }
      }
    }
  },
  methods: {
    vote () {
      if (this.userLoggedIn()) {
        if (this.upvoted) {
          this.downvote()
        } else {
          this.upvoteActive = !this.upvoteActive
        }
      }
    },
    upvote () {
      this.voting = true
      try {
        sc2.setAccessToken(this.$store.state.accessToken)
        sc2.vote(this.$store.state.username, this.gigData.author, this.gigData.permlink, parseInt(this.upvoteRange) * 100, (res) => {
          this.upvoteActive = false
          this.gigData.active_votes.push({voter: this.$store.state.username, weight: parseInt(this.upvoteRange)})
          this.$notify({
            title: 'Success',
            message: 'Your vote has been cast successfully',
            type: 'success'
          })
        })
      } catch (err) {
        this.$notify.error({
          title: 'Error',
          message: `There was an error voting on your post. Error details - ${err}`
        })
        this.voting = false
      }
    }
  },
  downvote () {
    this.unvoting = true
    try {
      sc2.setAccessToken(this.$store.state.accessToken)
      sc2.vote(this.$store.state.username, this.gigData.author, this.gigData.permlink, 0, (res) => {
        this.unvoting = false
        this.gigData.active_votes = this.gigData.active_votes.filter((x) => x.voter !== this.$store.state.username)
        this.$notify({
          title: 'Success',
          message: 'You have unvoted this post successfully',
          type: 'success'
        })
      })
    } catch (err) {
      this.$notify.error({
        title: 'Error',
        message: `There was an error voting on your post. Error details - ${err}`
      })
      this.unvoting = false
    }
  }
}
</script>

<style lang="scss">
$blue: #6361D0;
  a.upvote-btn {
    display: inline-block;
    padding: 0px 4px;
    border: 1px solid gray;
    border-radius: 50%;
    font-size: .8rem;
    &.upvoted {
      border-radius: 1px solid $blue;
      background-color: $blue;
      i {
        color: white;
      }
    }
    i {
      color: gray;
    }
  }
.card.gig {
  cursor: pointer;
  .card-image {
    min-height: 10em;
  }
  .card-content {
    min-height: 11em;
    .sellerPic {
      border-radius: 50%;
      border: 1px solid #6361D0;
      width: 2.1em;
      height: 2.1em;
      margin-top: 0px;
      margin-left: 0em;
      display: inline-block;
    }
    .price {
      margin-top: 0.5em;
      font-weight: 500;
    }
  }
  .card-action {
    padding: 1em;
    a:not(.btn):not(.btn-large):not(.btn-large):not(.btn-floating) {
      color: #9e9e9e;
      margin-right: 10px;
      cursor: pointer;
    }
  }
}
</style>
