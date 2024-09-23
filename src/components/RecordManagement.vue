<template>
<div class="container">
  <div class="form-group">
    <label for="main-profession">主职业:</label>
    <select v-model="mainProfession" id="main-profession">
      <option v-for="profession in professions" :key="profession" :value="profession">
        {{ profession }}
      </option>
    </select>

    <label for="secondary-profession">副职业:</label>
    <select v-model="secondaryProfession" id="secondary-profession">
      <option v-for="profession in professions" :key="profession" :value="profession">
        {{ profession }}
      </option>
    </select>
  </div>

  <div class="form-group">
    <label for="wins">胜场 (0-12):</label>
    <input v-model="wins" type="number" id="wins" min="0" max="12" />

    <label for="losses">负场 (0-3):</label>
    <input v-model="losses" type="number" id="losses" min="0" max="3" />
  </div>

  <div class="form-group">
    <label for="avg-win">均胜:</label>
    <input v-model="avgWin" id="avg-win" readonly />
  </div>

  <div class="form-group">
    <button @click="addRecord">添加</button>
    <button @click="calculateAvgWin">计算均胜</button>
  </div>

  <table>
    <thead>
      <tr>
        <th>轮次</th>
        <th>职业</th>
        <th>战绩 (胜-负)</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(record, index) in paginatedRecords" :key="index">
        <td>{{ currentPage * pageSize + index + 1 }}</td>
        <td>{{ record.profession }}</td>
        <td>{{ record.performance }}</td>
      </tr>
    </tbody>
  </table>

  <div class="pagination">
    <button :disabled="currentPage === 0" @click="prevPage">上一页</button>
    <button :disabled="nextPageDisabled" @click="nextPage">下一页</button>
  </div>
</div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

export default defineComponent({
name: 'RecordManagement',
setup() {
  // 初始化数据
  const professions = ref<string[]>(Array.from({ length: 11 }, (_, i) => (i + 1).toString()));
  const mainProfession = ref<string>('');
  const secondaryProfession = ref<string>('');
  const wins = ref<number | null>(null);
  const losses = ref<number | null>(null);
  const results = ref<{ wins: number; profession: string; performance: string }[]>([]);
  const avgWin = ref<string>('');
  const pageSize = 20;
  const currentPage = ref<number>(0);

  // 验证输入的胜场和负场
  const validateScores = (): boolean => {
    if (wins.value === null || losses.value === null) {
      alert('请输入胜场和负场');
      return false;
    }
    if (wins.value < 0 || wins.value > 12) {
      alert('胜场必须在 0 到 12 之间');
      return false;
    }
    if (losses.value < 0 || losses.value > 3) {
      alert('负场必须在 0 到 3 之间');
      return false;
    }
    return true;
  };

  // 验证主职业和副职业是否相同
  const validateProfessions = (): boolean => {
    if (mainProfession.value === secondaryProfession.value) {
      alert('主职业和副职业不能相同');
      return false;
    }
    return true;
  };

  // 计算均胜
  const calculateAvgWin = () => {
    if (results.value.length < 30) {
      alert('记录不足 30 轮');
      return;
    }
    const totalWins = results.value.slice(0, 30).reduce((sum, record) => sum + record.wins, 0);
    avgWin.value = (totalWins / 30).toFixed(2);
  };

  // 添加记录
  const addRecord = () => {
    if (!validateProfessions() || !validateScores()) {
      return;
    }

    results.value.push({
      wins: wins.value!,
      profession: mainProfession.value,
      performance: `${wins.value}-${losses.value}`,
    });

    /* // 清空表单
    mainProfession.value = '';
    secondaryProfession.value = '';
    wins.value = null;
    losses.value = null; */
  };

  // 分页记录
  const paginatedRecords = computed(() => {
    const startIndex = currentPage.value * pageSize;
    return results.value.slice(startIndex, startIndex + pageSize);
  });

  // 下一页是否禁用
  const nextPageDisabled = computed(() => {
    return (currentPage.value + 1) * pageSize >= results.value.length;
  });

  // 下一页
  const nextPage = () => {
    currentPage.value++;
  };

  // 上一页
  const prevPage = () => {
    currentPage.value--;
  };

  return {
    professions,
    mainProfession,
    secondaryProfession,
    wins,
    losses,
    avgWin,
    paginatedRecords,
    currentPage,
    nextPageDisabled,
    addRecord,
    calculateAvgWin,
    nextPage,
    prevPage,
  };
},
});
</script>

<style scoped>
.container {
padding: 20px;
}
.form-group {
margin-bottom: 15px;
}
table {
width: 100%;
border-collapse: collapse;
}
table th, table td {
border: 1px solid #ddd;
padding: 8px;
}
.pagination {
margin-top: 10px;
}
button {
margin-right: 10px;
}
</style>
