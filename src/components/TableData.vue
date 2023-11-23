<template>
  <table class="mb-2 table">
    <thead>
      <tr>
        <th id="position_id" data-sorted="1" @click="sortNumber('o_id', $event)">ID</th>
        <th id="client_name" data-sorted="1" @click="sortString('client_name', $event)">ФИО клиента</th>
        <th id="diet" data-sorted="1" @click="sortArrayString('diets', $event)">Диеты</th>
        <th id="tarif" data-sorted="1"  @click="sortArrayString('tariff', $event)">Тарифы</th>
        <th id="adres" data-sorted="1" @click="sortString('address', $event)">Адрес</th>
        <th id="phone" data-sorted="1" @click="sortNumber('phone', $event)">Телефон</th>
        <th id="date_info" data-sorted="1" @click="sortArrayObject('dates', $event)">Даты</th>
        <th id="discoint" data-sorted="1" @click="sortNumber('discount', $event)">Скидка</th>
        <th id="total_sum" data-sorted="1" @click="sortNumber('order_sum', $event)">Cумма заказа</th>
        <th id="payed_sum" data-sorted="1" @click="sortNumber('order_payed', $event)">Оплачено</th>
        <th id="status_pay" data-sorted="1" @click="sortString('pay_status', $event)">Статус оплаты</th>
        <th id="comment_courier" data-sorted="1" @click="sortString('courier_comment', $event)">Комментарии куръера</th>
        <th id="inner_comment" data-sorted="1" @click="sortString('inner_comment', $event)">Внутренние комментарии</th>
        <th id="status_diet" :class="{ 'text-red': showSortStatus }" @click="handleSortClick">
          Статус диеты
          <ul v-if="showSortStatus">
            <li @click.stop="sortDaysBegin">Начинается через Х дней</li>
            <li @click.stop="sortDaysToday">Заканчивается сегодня</li>
            <li @click.stop="sortDaysTomorrow">Заканчивается завтра</li>
            <li @click.stop="sortDaysEnd">Заканчивается через Х дней</li>
            <li @click.stop="sortDaysPast">Закончилась Х дней назад</li>
          </ul>
        </th>
      </tr>
    </thead>
    <tbody>
        <tr
          v-for="(position, index) in processedJson"
          :key="position.o_id"
          :class="{ 'bg': index % 2 !== 0 }"
        >
            <td class="position-id">{{ position.o_id }}</td>
            <td>{{ position.client_name }}</td>
            <td class="diet">
              <span
                v-for="(diet, dietIndex) in position.diets"
                :key="dietIndex"
              >
                {{ diet }} <br>
                <span v-if="dietIndex < position.diets.length - 1" class="line"></span>  
              </span>
            </td>
            <td class="tarif">
              <span
                v-for="(tarif, tarifIndex) in position.tariff"
                :key="tarifIndex"
              >
                {{ tarif }} <br>
                <span v-if="tarifIndex < position.tariff.length - 1" class="line"></span>  
              </span>
            </td>
            <td>{{ position.address }}</td>
            <td>{{ position.phone }}</td>
            <td class="date">
              <span
                v-for="(date, dateIndex) in position.dates"
                :key="dateIndex"
              >
                {{ formatDate(date.start_date) }}&nbsp;-&nbsp;{{ formatDate(date.end_date) }} <br>
                <span v-if="dateIndex < position.dates.length - 1" class="line"></span>  
              </span>
            </td>
            <td class="discount">{{ position.discount }}</td>
            <td>{{ position.order_sum }} р.</td>
            <td>{{ position.order_payed }} р.</td>
            <td :class="{ 'payed': position.pay_status === 'Оплачен', 'no-payed': position.pay_status !== 'Оплачен' }">
              {{ position.pay_status }}
            </td>
            <td class="courier-comment">{{ position.courier_comment }}</td>
            <td class="inner-comment">{{ position.inner_comment }}</td>
            
            <td class="status">
              <span
                v-for="(date, dateIndex) in position.dates"
                :key="dateIndex"
              >
              {{ letDays(date.start_date, date.end_date) }} <br>
              <span v-if="dateIndex < position.dates.length - 1" class="line"></span>  
              </span>
            </td>
        </tr>
    </tbody>
  </table>
</template>

<script>
import { ref, onMounted } from 'vue';
import json from '@/assets/data.json';

export default {
  name: 'TableData',
  setup() {
    const myJson = ref(json);
    const processedJson = ref();
    const showSortStatus = ref(false);
    const currentDate = new Date();
    const oneDayMilliseconds = 24 * 60 * 60 * 1000;

    onMounted(() => {
      processedJson.value = [];
      processedJson.value = myJson.value;
      console.log(processedJson.value);
    });

    const formatDate = (dateString) => {
      const dateObject = new Date(dateString);
      const day = dateObject.getDate();
      const month = dateObject.getMonth() + 1;
      const formattedDay = day < 10 ? `0${day}` : day;
      const formattedMonth = month < 10 ? `0${month}` : month;
      const formattedDate = `${formattedDay}.${formattedMonth}`;
      return formattedDate;
    };

    const changeDaysWord = (daysWord, daysRemaining) => {
      let resultWord = daysWord;

      if (daysRemaining % 10 === 1 && daysRemaining % 100 !== 11) {
        resultWord = 'день';
      } else if (
        (daysRemaining % 10 === 2 || daysRemaining % 10 === 3 || daysRemaining % 10 === 4) &&
        (daysRemaining % 100 < 10 || daysRemaining % 100 >= 20)
      ) {
        resultWord = 'дня';
      }

      return resultWord;
    };
    
    const letDays = (start_date, end_date) => {
      const startDate = new Date(start_date);
      const endDate = new Date(end_date);
      let daysWord = 'дней';

      currentDate.setHours(0, 0, 0, 0);
      startDate.setHours(0, 0, 0, 0);
      endDate.setHours(0, 0, 0, 0);

      if (currentDate <= startDate) {
        const daysRemaining = calculateDays(startDate, currentDate);
        if (daysRemaining === 0) {
          return 'Начинается сегодня';
        } else {
          return `Начинается через ${daysRemaining} ${changeDaysWord(daysWord, daysRemaining)}`;
        }
      }

      if (currentDate > endDate) {
        const daysPassed = calculateDays(currentDate, endDate);
        return `Закончилась ${daysPassed} ${changeDaysWord(daysWord, daysPassed)} назад`;
      }

      if (currentDate > startDate && currentDate <= endDate) {
        const daysRemaining = calculateDays(endDate, currentDate);
        if (daysRemaining === 0) {
          return 'Заканчивается сегодня';
        } else if (daysRemaining === 1) {
          return 'Заканчивается завтра';
        } else {
          return `Заканчивается через ${daysRemaining} ${changeDaysWord(daysWord, daysRemaining)}`;
        }
      }

      return 'Ошибка в датах';
    }

    const sortNumber = (property, event) => {
      const sortOrder = parseInt(event.currentTarget.getAttribute('data-sorted'), 10);

      processedJson.value = myJson.value.sort(
        (a, b) => sortOrder * (a[property] - b[property])
      );
      event.currentTarget.setAttribute('data-sorted', -sortOrder);
    };

    const sortString = (property, event) => {
      let sortOrder = parseInt(event.currentTarget.getAttribute('data-sorted'), 10);

      const arrayHave = [];
      const arrayEmpty = [];

      myJson.value.forEach(item => {
        if (item[property] !== "" && item[property] !== null) {
          arrayHave.push(item);
        } else {
          arrayEmpty.push(item);
        }
      });

      arrayHave.sort((a, b) => {
        const aValue = a[property].toLowerCase();
        const bValue = b[property].toLowerCase();

        return sortOrder * aValue.localeCompare(bValue);
      });

      const sortedAndMergedArray = arrayHave.concat(arrayEmpty);
      processedJson.value = sortedAndMergedArray;

      event.currentTarget.setAttribute('data-sorted', -sortOrder);
    };

    const sortArrayString = (property, event) => {
      let sortOrder = parseInt(event.currentTarget.getAttribute('data-sorted'), 10);

      processedJson.value = myJson.value.sort((a, b) => {
        const aValue = a[property][0].toLowerCase();
        const bValue = b[property][0].toLowerCase();

        return sortOrder * aValue.localeCompare(bValue);
      });

      event.currentTarget.setAttribute('data-sorted', -sortOrder);
    };

    const sortArrayObject = (property, event) => {
      let sortOrder = parseInt(event.currentTarget.getAttribute('data-sorted'), 10);

      processedJson.value = myJson.value.sort((a, b) => {
        const aValue = formatDate(a[property][0].start_date).toLowerCase();
        const bValue = formatDate(b[property][0].start_date).toLowerCase();

        return sortOrder * aValue.localeCompare(bValue);
      });

      event.currentTarget.setAttribute('data-sorted', -sortOrder);
    };

    const handleSortClick = () => {
      showSortStatus.value = !showSortStatus.value;
    };

    const sortDaysBegin = () => {
      const positiveDifference = [];

      myJson.value.forEach(item => {
        const daysDifference = calculateDays(new Date(item.dates[0].start_date), currentDate);

        if (daysDifference >= 0) {
          positiveDifference.push(item);
        }
      });

      positiveDifference.sort((a, b) => {
        const daysDifferenceA = calculateDays(new Date(a.dates[0].start_date), currentDate);
        const daysDifferenceB = calculateDays(new Date(b.dates[0].start_date), currentDate);

        return daysDifferenceA - daysDifferenceB;
      });

      processedJson.value = positiveDifference;
    };

    const sortDaysToday = () => {
      const positive = [];

      myJson.value.forEach(item => {
        if (currentDate > new Date(item.dates[0].start_date) && currentDate <= new Date(item.dates[0].end_date)) {
          const daysRemaining = calculateDays(new Date(item.dates[0].end_date), currentDate);
          if (daysRemaining === 0) {
            positive.push(item);
          }
        }
      });

      if (positive.length === 0 ) {
        return alert('Диет, заканчивающихся сегодня нет');
      }

      processedJson.value = positive;
    };

    const sortDaysTomorrow = () => {
      const positive = [];

      myJson.value.forEach(item => {
        if (currentDate > new Date(item.dates[0].start_date) && currentDate <= new Date(item.dates[0].end_date)) {
          const daysRemaining = calculateDays(new Date(item.dates[0].end_date), currentDate);
          if (daysRemaining === 1) {
            positive.push(item);
          }
        }
      });

      if (positive.length === 0 ) {
        return alert('Диет, заканчивающихся завтра нет');
      }

      processedJson.value = positive;
    };

    const sortDaysEnd= () => {
      const positive = [];

      myJson.value.forEach(item => {
        if (currentDate > new Date(item.dates[0].start_date) && currentDate <= new Date(item.dates[0].end_date)) {
          const daysRemaining = calculateDays(new Date(item.dates[0].end_date), currentDate);
          if (daysRemaining >= 0) {
            positive.push(item);
          }
        }
      });

      positive.sort((a, b) => {
        const daysDifferenceA = calculateDays(new Date(a.dates[0].end_date), currentDate);
        const daysDifferenceB = calculateDays(new Date(b.dates[0].end_date), currentDate);

        return daysDifferenceA - daysDifferenceB;
      });

      processedJson.value = positive;
    };

    const sortDaysPast = () => {
      const positiveDifference = [];
      // const negativeDifference = [];

      // myJson.value.forEach(item => {
      //   const daysDifference = calculateDays(currentDate, new Date(item.dates[0].end_date));

      //   if (daysDifference > 0) {
      //     positiveDifference.push(item);
      //   } else {
      //     // negativeDifference.push(item);
      //   }
      // });

      myJson.value.forEach(item => {
        let positiveDifferenceFound = false;

        item.dates.forEach(dateItem => {
          if (positiveDifferenceFound) {
              return;
          }
          const daysDifference = calculateDays(currentDate, new Date(dateItem.end_date));

          if (daysDifference > 0) {
            positiveDifference.push(item);
            positiveDifferenceFound = true;
            return;
          }
        });
      });

      positiveDifference.sort((a, b) => {
        const daysDifferenceA = calculateDays(currentDate, new Date(a.dates[0].end_date));
        const daysDifferenceB = calculateDays(currentDate, new Date(b.dates[0].end_date));

        return daysDifferenceA - daysDifferenceB;
      });

      // const sortedAndMergedArray = positiveDifference.concat(negativeDifference);
      processedJson.value = positiveDifference;
    };

    const calculateDays = (dateA, dateB) => {
      const timeDifference = dateA - dateB;
      return Math.round(timeDifference / oneDayMilliseconds);
    };

    return {
      processedJson,
      formatDate,
      letDays,
      sortNumber,
      sortString,
      sortArrayString,
      sortArrayObject,
      showSortStatus,
      handleSortClick,
      sortDaysBegin,
      sortDaysToday,
      sortDaysPast,
      sortDaysTomorrow,
      sortDaysEnd
    };
  },
};
</script>

<style scoped>
table {
  border-collapse: collapse;
  width: 100%;
}
table th, table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
table th {
  text-align: center;
  background-color: #ebf5e5;
}
table tr.bg {
  background-color: #f2f2f2;
}
table td.payed {
  text-align: center;
  background-color: #b3f08e;
}
table td.no-payed {
  text-align: center;
  background-color: #f08e8e;
}
tr td.status {
  background-color: #dfd4f6;
}
tr.bg td.status {
  background-color: #c2a8f7;
}
.line {
  display: block;
  border-bottom: 1px solid #000;
  margin: 10px 0;
}
.courier-comment {
  max-width: 200px;
}
.inner-comment {
  max-width: 350px;
}
.position-id,
.diet,
.tarif,
.date,
.discount {
  text-align: center;
}
.position-id {
  background-color: #7fffd4;
}
thead th {
  cursor: pointer;
}
thead th:hover {
  color: red;
}
#status_diet {
  position: relative;
}
ul {
  list-style: none;
  position: absolute;
  top: -10px;
  right: 100%;
  background-color: #fff;
  padding: 10px;
  border-radius: 8px;
  white-space: nowrap;
  text-align: start;
  line-height: 130%;
  box-shadow: 0 0 8px rgba(0,0,0,.15);
  color: #000;
}
ul li:hover {
  color: #0009ff;
}
.text-red {
  color: red;
}
</style>
