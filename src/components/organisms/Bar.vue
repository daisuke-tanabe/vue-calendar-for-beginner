<template>
  <div :class="$style.wrapper">
    <div :class="$style.pageResetButton">
      <VButton text="今日" @on-click="onClickResetButton" />
    </div>

    <div :class="$style.pager">
      <button :class="$style.pager__button" type="button" @click="onClickPreviousPage">＜</button>
      <button :class="$style.pager__button" type="button" @click="onClickNextPage">＞</button>
    </div>

    <div :class="$style.date">{{ date }}</div>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex';
import VButton from '~atoms/VButton.vue';

export default {
  components: {
    VButton,
  },

  computed: {
    ...mapState({
      year: 'year',
      month: 'month',
    }),
    date: ({ year, month }) => `${year}年${month}月`,
  },

  methods: {
    ...mapActions({
      nextPage: 'nextPage',
      previousPage: 'previousPage',
      resetPage: 'resetPage',
    }),
    onClickNextPage() {
      this.nextPage();
    },
    onClickPreviousPage() {
      this.previousPage();
    },
    onClickResetButton() {
      this.resetPage();
    },
  },
};
</script>

<style lang="scss" module>
.wrapper {
  align-items: center;
  border-bottom: 1px solid #dadce0;
  display: flex;
  padding: 16px;
}

// pageResetButton
// ------------------------------
.pageResetButton {
  flex: 0 1 64px;
  width: 64px;
}

// pager
// ------------------------------
.pager {
  flex: 0 1 auto;
  margin-left: 32px;
}

.pager__button {
  background: none;
  border: none;
  font-size: 16px;
  font-weight: bold;
  outline: none;
  padding-top: 3px;

  &:hover {
    cursor: pointer;
  }
}

// date
// ------------------------------
.date {
  flex: 0 1 auto;
  font-size: 20px;
  margin-left: 20px;
}
</style>
