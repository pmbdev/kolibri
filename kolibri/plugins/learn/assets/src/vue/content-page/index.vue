<template>

  <div>

    <page-header :title="content.title"></page-header>

    <content-renderer
      v-show="!searchOpen"
      class="content-renderer"
      :id="content.id"
      :kind="content.kind"
      :files="content.files"
      :content-id="content.content_id"
      :channel-id="channelId"
      :available="content.available"
      :extra-fields="content.extra_fields">
    </content-renderer>

    <icon-button v-link="nextContentLink" v-if="progress >= 1 && showNextBtn" class="next-btn">
    {{ $tr("nextContent") }}
    <svg class="right-arrow" src="../icons/arrow_right.svg"></svg></icon-button>

    <p class="page-description">{{ content.description }}</p>

    <download-button v-if="canDownload" :files="content.files"></download-button>

    <expandable-content-grid
      class="recommendation-section"
      v-if="pageMode === Constants.PageModes.LEARN"
      :title="recommendedText"
      :contents="recommended">
    </expandable-content-grid>

  </div>

</template>


<script>

  const Constants = require('../../state/constants');
  const getters = require('../../state/getters');
  const ContentNodeKinds = require('kolibri.coreVue.vuex.constants').ContentNodeKinds;
  const UserKinds = require('kolibri.coreVue.vuex.constants').UserKinds;

  module.exports = {
    $trNameSpace: 'learnContent',
    $trs: {
      recommended: 'Recommended',
      nextContent: 'Next Content',
    },
    computed: {
      Constants() {
        return Constants; // allow constants to be accessed inside templates
      },
      canDownload() {
        if (this.content) {
          // computed property sometimes runs before the store is ready.
          return this.content.kind !== ContentNodeKinds.EXERCISE;
        }
        return false;
      },
      showNextBtn() {
        if (this.content) {
          return this.content.kind === ContentNodeKinds.EXERCISE;
        }
        return false;
      },
      recommendedText() {
        return this.$tr('recommended');
      },
      progress() {
        if (this.userkind.includes(UserKinds.LEARNER)) {
          return this.summaryProgress;
        }
        return this.sessionProgress;
      },
      nextContentLink() {
        if (this.content.next_content.kind !== ContentNodeKinds.TOPIC) {
          return {
            name: this.pagename,
            params: { id: this.content.next_content.id },
          };
        }
        return {
          name: Constants.PageNames.EXPLORE_TOPIC,
          params: { id: this.content.next_content.id },
        };
      },
    },
    components: {
      'page-header': require('../page-header'),
      'expandable-content-grid': require('../expandable-content-grid'),
    },
    vuex: {
      getters: {
        // general state
        pageMode: getters.pageMode,

        // TODO - remove hack
        // temporarily using this to address an IE10 bug where the PDF
        // renderer displayed on top of the search pane.
        // see https://trello.com/c/LSevcA40/263-windows-7-ie-10-when-you-click-the-search-button-on-a-pdf-page-under-learn-tab-the-pdf-file-still-shows-when-it-shouldnt
        searchOpen: state => state.searchOpen,

        // attributes for this content item
        content: (state) => state.pageState.content,
        channelId: (state) => state.core.channels.currentId,
        pagename: (state) => state.pageName,

        // only used on learn page
        recommended: (state) => state.pageState.recommended,

        summaryProgress: (state) => state.core.logging.summary.progress,
        sessionProgress: (state) => state.core.logging.session.progress,
        userkind: (state) => state.core.session.kind,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.coreTheme'

  .recommendation-section
    margin-top: 4em

  .next-btn
    float: left
    background-color: #4A8DDC
    border-color: #4A8DDC
    color: $core-bg-light
    padding-left: 16px
    padding-right: 6px
    padding-bottom: 0

  .next-btn:hover svg
    fill: $core-bg-light

  .right-arrow
    fill: $core-bg-light

  .right-arrow:hover
    fill: $core-bg-light

</style>

