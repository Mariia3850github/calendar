<script lang="ts" setup>
const currentDate = ref(new Date());
const currentYear = ref(currentDate.value.getFullYear());
const currentMonth = ref(currentDate.value.getMonth() + 1);
const weeksInMonth = ref(0);
const calendar: any = ref();
const dateSelected: any = ref(null);
const currentDay = ref(currentDate.value.getDate());
// const firstDayInMonth = ref(new Date())

const languageList = ref(["ru", "eng"])
const selectedLanguage = ref("ru")

function getFirstDayOfMonth(year: number, month: number) {
  // Создаем объект Date для первого дня месяца
  const firstDay = new Date(year, month, 1);
  // Получаем номер дня недели (0 для воскресенья, 1 для понедельника и т.д.)
  return firstDay.getDay();
}

function getDaysInMonth(year: number, month: number) {
  return new Date(year, month, 0).getDate();
}

const monthNames = [
  { rus: "Январь", eng: "January" },
  { rus: "Февраль", eng: "February" },
  { rus: "Март", eng: "March" },
  { rus: "Апрель", eng: "April" },
  { rus: "Май", eng: "May" },
  { rus: "Июнь", eng: "June" },
  { rus: "Июль", eng: "July" },
  { rus: "Август", eng: "August" },
  { rus: "Сентябрь", eng: "September" },
  { rus: "Октябрь", eng: "October" },
  { rus: "Ноябрь", eng: "November" },
  { rus: "Декабрь", eng: "December" },
];

const mnthNames = [
  { rus: "Янв", eng: "Jan" },
  { rus: "Фев", eng: "Feb" },
  { rus: "Мар", eng: "Mar" },
  { rus: "Апр", eng: "Apr" },
  { rus: "Май", eng: "May" },
  { rus: "Июн", eng: "Jun" },
  { rus: "Июл", eng: "Jul" },
  { rus: "Авг", eng: "Aug" },
  { rus: "Сен", eng: "Sep" },
  { rus: "Окт", eng: "Oct" },
  { rus: "Ноя", eng: "Nov" },
  { rus: "Дек", eng: "Dec" },
];

const dayNames = [
  { rus: "Пн", eng: "Mon" },
  { rus: "Вт", eng: "Tue" },
  { rus: "Ср", eng: "Wed" },
  { rus: "Чт", eng: "Thu" },
  { rus: "Пт", eng: "Fri" },
  { rus: "Сб", eng: "Sat" },
  { rus: "Вс", eng: "Sun" },
];

async function prevMonth() {
  if (currentMonth.value == 0) {
    currentMonth.value = 12;
    currentYear.value--;
  } else {
    currentMonth.value = currentMonth.value - 1;
  }

  calendar.value = await calendarGeneration();
}

async function nextMonth() {
  if (currentMonth.value == 12) {
    currentMonth.value = 1;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }

  calendar.value = await calendarGeneration();
}

onMounted(async () => {
  weeksInMonth.value = Math.ceil(
    getDaysInMonth(currentYear.value, currentMonth.value) / 7
  );
  calendar.value = await calendarGeneration();
});

async function calendarGeneration() {
  let calendar = [];

  const firstDayOfMonth = getFirstDayOfMonth(
    currentYear.value,
    currentMonth.value - 1
  );

  const daysInPrevMonth = getDaysInMonth(
    currentYear.value,
    currentMonth.value - 1
  );
  const daysInMonth = getDaysInMonth(currentYear.value, currentMonth.value);

  const daysInNextMonth =
    7 * weeksInMonth.value - (daysInPrevMonth - firstDayOfMonth + daysInMonth);

  let dayCounter = 1;

  for (let i = 0; i < weeksInMonth.value; i++) {
    let week = [];

    for (let j = 1; j <= 7; j++) {
      if (i === 0 && j < firstDayOfMonth) {
        if (currentMonth.value == 1) {
          // Добавляем числа из предыдущего месяца
          const dayFromPrevMonth = daysInPrevMonth - (firstDayOfMonth - j) + 1;
          week.push({
            year: currentYear.value - 1,
            month: 12,
            day: dayFromPrevMonth,
          });
        } else {
          // Добавляем числа из предыдущего месяца
          const dayFromPrevMonth = daysInPrevMonth - (firstDayOfMonth - j) + 1;
          week.push({
            year: currentYear.value,
            month: currentMonth.value - 1,
            day: dayFromPrevMonth,
          });
        }
      } else if (dayCounter <= daysInMonth) {
        week.push({
          year: currentYear.value,
          month: currentMonth.value,
          day: dayCounter,
        });
        dayCounter++;
      } else {
        if (currentMonth.value == 12) {
          // Добавляем числа из следующего месяца
          week.push({
            year: currentYear.value + 1,
            month: 1,
            day: dayCounter - daysInMonth,
          });
          dayCounter++;
        } else {
          // Добавляем числа из следующего месяца
          week.push({
            year: currentYear.value,
            month: currentMonth.value + 1,
            day: dayCounter - daysInMonth,
          });
          dayCounter++;
        }
      }
    }

    calendar.push(week);
  }

  return calendar;
}

// async function weeksInMonth () {
// return getDaysInMonth(currentYear.value, currentMonth.value) / 7
// }

watch(currentMonth, async () => {
  weeksInMonth.value = Math.ceil(
    getDaysInMonth(currentYear.value, currentMonth.value) / 7
  );
});

async function selectDate(year: number, day: number, month: number) {
  currentMonth.value = month;
  currentYear.value = year;

  const selectedDate = new Date(year, month - 1, day);
  dateSelected.value = selectedDate;

  calendar.value = await calendarGeneration();
  currentDay.value = day;
}

const isActive = (year: number, month: number, day: number) => {
  // Добавьте здесь логику для определения, является ли день активным
  // Например, сравнение с текущей датой или другой вашей логики
  return (
    year === currentYear.value &&
    month === currentMonth.value &&
    day === currentDay.value
  );
};

async function toggleLanguage () {
  if (selectedLanguage.value === 'ru') {
    selectedLanguage.value = 'eng' 
  } else {
    selectedLanguage.value = 'ru' 
  }
}
</script>
<template>
  <div>
    <h2>Календарь</h2>

    <div style="margin-bottom: 1.5rem;">
      День недели: {{ currentDate.getDay() }} <br />
      Текущий месяц: {{ currentMonth }} <br />
      Год: {{ currentYear }} <br />
      Число месяца: {{ currentDay }} <br />
      Дней в месяце: {{ getDaysInMonth(currentYear, currentMonth) }} <br />
      Недель в месяце: {{ weeksInMonth }}
    </div>

    <div id="calendar">
      <div class="heading">
        <button @click="prevMonth()" class="arrow arrow-left">
          <i class="bi bi-caret-left-fill"></i>
        </button>
        <p>
          {{ currentDay }}, 
          <span v-if="selectedLanguage === 'eng'">{{ monthNames[currentMonth - 1].eng }}</span>
          <span v-if="selectedLanguage === 'ru'">{{ monthNames[currentMonth - 1].rus }}</span>
          {{ currentYear }}
        </p>
        <button @click="nextMonth()" class="arrow arrow-right">
          <i class="bi bi-caret-right-fill"></i>
        </button>
      </div>
      <div class="dayName">
        <ul>
          <li v-for="(dayName, index) in dayNames" :key="index">
            <span v-if="selectedLanguage === 'eng'">{{ dayName.eng }}</span>
            <span v-if="selectedLanguage === 'ru'">{{ dayName.rus }}</span>
          </li>
        </ul>
      </div>
      <div class="body">
        <div class="week" v-for="(week, index) in calendar" :key="index">
          <div class="day" v-for="(element, index) in week" :key="index">
            <button
              @click="selectDate(element.year, element.day, element.month)"
              :class="{
                active: isActive(element.year, element.month, element.day),
              }"
            >
              <span>{{ element.day }}</span>
            </button>
          </div>
        </div>
      </div>
      <div>
        <p style="margin: 1rem 0">{{ dateSelected }}</p>
      </div>
      <div><button class="languageToggler" @click="toggleLanguage()">Сменить язык</button></div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.languageToggler {
  border: 0;
  background-color: #f7f7f7;
  // border: 1px solid #3ef;
  padding: .5rem 2rem;
  border-radius: 5px;
  transition: 400ms;
  transform: scale(1);
  &:active {
    transform: scale(.95);
  }
}
.week {
  display: flex;
  .day {
    padding: 5px;
    text-align: center;
    button {
      display: flex;
      height: 35px;
      width: 35px;
      background-color: #fff;
      border: 1px solid transparent;
      transition: 400ms;
      &.active {
        // background-color: #eee;
        border-color: #3ef;
      }
      span {
        display: block;
        margin: auto;
      }
      &:hover {
        // background-color: #eee;
        border-color: #3ef;
      }
    }
  }
}
#calendar {
  width: 300px;
}
.heading {
  display: flex;
  width: 100%;
  justify-content: space-between;
  padding: 0 15px;  
  p {
    display: block;
    margin: auto 0;
  }
  button {
    border: 0;
    background-color: transparent;
    padding: 0;
  }
}
.dayName {
  ul {
    display: flex;
    padding: 0;
    margin: 1rem 0 .5rem;
    list-style-type: none;
    justify-content: space-around;
    li {
      //   padding: 5px;s
      padding: 0 15px;
      width: 45px;
    }
  }
}
</style>
