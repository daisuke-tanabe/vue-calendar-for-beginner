<template>
  <div ref="schedule" :class="$style.calendar">
    <div :class="$style.calendar__container">
      <CalendarCell
        v-for="schedule in schedules"
        :key="schedule.date"
        :today="today"
        :schedule="schedule"
        @on-click-cell="showModal"
        @on-click-remove-task-button="removeTask"
        @on-click-task-overview-button="scaleCell"
      />
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters, mapState } from 'vuex';
import WEEKS from '~/constants/weeks';
import CalendarCell from '~organisms/CalendarCell.vue';
import zeroPadding from '~utilities/zeroPadding';

// 1月に最大42日分表示する（6週間 * 7 = 42）
const MAX_DAYS_OF_MONTH = 42;

function makeSchedule({ year, month, days, current, holiday, tasks }) {
  return days.map((day) => {
    const date = `${year}-${zeroPadding(month)}-${zeroPadding(day)}`;
    const filteredByDateTask = tasks.filter((task) => task.date === date);
    return { day, current, date, holiday: holiday[date] || '', tasks: filteredByDateTask };
  });
}

function getOverviews() {
  return [...Array(MAX_DAYS_OF_MONTH)].map(() => false);
}

export default {
  components: { CalendarCell },

  data() {
    return {
      weeks: WEEKS,
      overviews: getOverviews(),
    };
  },

  computed: {
    ...mapGetters(['today']),
    ...mapState(['year', 'month', 'day', 'holiday', 'tasks']),
    // 今月、先月、翌月のスケジュールをマージしたもの
    schedules: ({ year, month, holiday, tasks, weeks, overviews }) => {
      // 今月のスケジュール
      const scheduleMonth = makeSchedule({
        year,
        month,
        days: [...Array(new Date(year, month, 0).getDate()).keys()].map((index) => index + 1),
        tasks,
        current: true,
        holiday,
      });

      // 先月のはみだしスケジュール
      const weekOfFirstDay = new Date(year, month - 1, 1).getDay();
      const dayOfLatMonth = new Date(year, month - 1, 0).getDate();
      const scheduleLastMonth = makeSchedule({
        year,
        month: month - 1,
        days: [...Array(weekOfFirstDay).keys()].map((index) => dayOfLatMonth - index).reverse(),
        tasks,
        current: false,
        holiday,
      });

      // 翌月のはみだしスケジュール
      const firstDay = MAX_DAYS_OF_MONTH - scheduleMonth.length - scheduleLastMonth.length;
      const scheduleNextMonth = makeSchedule({
        year,
        month: month + 1,
        days: [...Array(firstDay).keys()].map((index) => index + 1),
        tasks,
        current: false,
        holiday,
      });

      return [...scheduleLastMonth, ...scheduleMonth, ...scheduleNextMonth].map((schedule, index) => ({
        ...schedule,
        firstWeek: index < weeks.length,
        overview: overviews[index],
        week: weeks[index % 7],
      }));
    },
  },

  created() {
    this.fetchHoliday();
  },

  mounted() {
    const TASKS = 'tasks';
    const localStorageTasks = localStorage.getItem(TASKS);
    if (localStorageTasks) {
      try {
        this.setTasks(JSON.parse(localStorageTasks));
      } catch (error) {
        // localStorageが壊れているなら削除
        localStorage.removeItem(TASKS);
      }
    }
    this.$store.watch(
      (state) => state.tasks,
      (value) => {
        localStorage.setItem(TASKS, JSON.stringify(value));
      }
    );
  },

  methods: {
    ...mapActions({
      fetchHoliday: 'fetchHoliday',
      setTargetDate: 'setTargetDate',
      setTasks: 'setTasks',
      toggleModal: 'toggleModal',
      removeTask: 'removeTask',
    }),
    scaleCell(date, $el) {
      const scheduleIndex = this.schedules.findIndex((schedule) => schedule.date === date);
      this.overviews = this.overviews.map((overview, index) => index === scheduleIndex);
      const bindEvent = {
        click: (event) => {
          if (!$el.contains(event.target)) {
            this.overviews = this.overviews.map(() => false);
            window.removeEventListener('click', bindEvent.click);
          }
        },
      };
      window.addEventListener('click', bindEvent.click);
    },
    showModal(date) {
      const isEnabledOverview = this.schedules.some((schedule) => schedule.overview);
      this.setTargetDate(date);
      this.toggleModal(!isEnabledOverview);
    },
  },
};
</script>

<style lang="scss" module>
.calendar {
  overflow: hidden;
  padding: 16px;
}

.calendar__container {
  background: #dadce0;
  border: 1px solid #dadce0;
  display: grid;
  gap: 1px;
  grid-template-columns: repeat(7, minmax(40px, 1fr));
  height: 100%;
}
</style>
