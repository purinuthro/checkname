<template>
    <div class="attendance-container">
      <h2>สรุปการเข้าเรียน</h2>
      <div class="navigation-buttons">
        <button @click="goToAttendance">ไปที่หน้าการเข้าเรียน</button>
      </div>
      <!-- ตัวเลือกสำหรับการดูสรุปรายสัปดาห์หรือรายเดือน -->
      <div class="form-group">
        <label>สรุปราย</label>
        <select v-model="summaryType" @change="fetchAttendanceSummary">
          <option value="weekly">สัปดาห์</option>
          <option value="monthly">เดือน</option>
        </select>
      </div>
  
      <!-- ช่องค้นหาชื่อนักเรียน -->
      <div class="form-group">
        <label>หาชื่อนักเรียน</label>
        <input 
          type="text" 
          v-model="searchQuery" 
          @input="filterBySearch" 
          placeholder="Enter student name" 
        />
      </div>
  
      <!-- แสดงผลข้อมูลสรุปรายสัปดาห์แยกตามนักเรียน -->
      <div v-if="summaryType === 'weekly'" class="summary-table">
        <h3>สรุปรายสัปดาห์</h3>
        <p>เริ่ม: {{ weekStart }} | จบ: {{ weekEnd }}</p>
        <div class="table-responsive">
          <table>
            <thead>
              <tr>
                <th>รหัสนักเรียน</th>
                <th>ชื่อจริง</th>
                <th>นามสุกล</th>
                <th>มาเรียน</th>
                <th>ขาด</th>
                <th>ลาป่วย</th>
                <th>ลากิจ</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(summary, studentId) in filteredWeeklySummary" :key="studentId">
                <td>{{ studentId }}</td>
                <td>{{ summary.firstName }}</td>
                <td>{{ summary.lastName }}</td>
                <td>{{ summary.attendClass }}</td>
                <td>{{ summary.missClass }}</td>
                <td>{{ summary.sickLeaveClass }}</td>
                <td>{{ summary.personalLeave }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
  
      <!-- แสดงผลข้อมูลสรุปรายเดือนแยกตามนักเรียน -->
      <div v-if="summaryType === 'monthly'" class="summary-table">
        <h3>สรุปรายเดือน {{ currentMonth }}</h3>
        <div class="table-responsive">
          <table>
            <thead>
              <tr>
                <th>วันที่</th>
                <th>รหัสนักเรียน</th>
                <th>ชื่อจริง</th>
                <th>นามสกุล</th>
                <th>มาเรียน</th>
                <th>ขาด</th>
                <th>ลาป่วย</th>
                <th>ลากิจ</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(entry, index) in filteredMonthlySummary" :key="index">
                <td>{{ entry.date }}</td>
                <td>{{ entry.studentID }}</td>
                <td>{{ entry.firstName }}</td>
                <td>{{ entry.lastName }}</td>
                <td>{{ entry.attendClass }}</td>
                <td>{{ entry.missClass }}</td>
                <td>{{ entry.sickLeaveClass }}</td>
                <td>{{ entry.personalLeave }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  import { useNuxtApp } from '#app'
  
  const { $supabase } = useNuxtApp()
  
  const summaryType = ref('weekly')
  const searchQuery = ref('')
  const currentMonth = new Date().toISOString().slice(0, 7)
  
  const weeklySummary = ref({})
  const monthlySummary = ref([])
  
  const filteredWeeklySummary = ref({})
  const filteredMonthlySummary = ref([])
  
  const getWeekStartAndEndDates = () => {
    const currentDate = new Date()
    const dayOfWeek = currentDate.getDay()
    const weekStart = new Date(currentDate)
    weekStart.setDate(currentDate.getDate() - dayOfWeek)
    const weekEnd = new Date(weekStart)
    weekEnd.setDate(weekStart.getDate() + 6)
    return {
      start: weekStart.toISOString().split('T')[0],
      end: weekEnd.toISOString().split('T')[0],
    }
  }
  
  const { start: weekStart, end: weekEnd } = getWeekStartAndEndDates()
  
  const fetchWeeklySummary = async () => {
    const { data, error } = await $supabase
      .from('attendance')
      .select(`
        studentID,
        attendClass,
        missClass,
        sickLeaveClass,
        personalLeave,
        students (
          firstName,
          lastName
        )
      `)
      .gte('date', weekStart)
      .lte('date', weekEnd)
  
    if (error) {
      console.error('Error fetching weekly summary:', error)
      return
    }
  
    weeklySummary.value = data.reduce((totals, entry) => {
      const { studentID, students, attendClass, missClass, sickLeaveClass, personalLeave } = entry
      if (!totals[studentID]) {
        totals[studentID] = {
          firstName: students.firstName,
          lastName: students.lastName,
          attendClass: 0,
          missClass: 0,
          sickLeaveClass: 0,
          personalLeave: 0,
        }
      }
      totals[studentID].attendClass += attendClass
      totals[studentID].missClass += missClass
      totals[studentID].sickLeaveClass += sickLeaveClass
      totals[studentID].personalLeave += personalLeave
      return totals
    }, {})
  
    filterBySearch()
  }
  
  const fetchMonthlySummary = async () => {
    const { data, error } = await $supabase
      .from('attendance')
      .select(`
        date,
        studentID,
        attendClass,
        missClass,
        sickLeaveClass,
        personalLeave,
        students (
          firstName,
          lastName
        )
      `)
      .like('date', `${currentMonth}%`)
  
    if (error) {
      console.error('Error fetching monthly summary:', error)
      return
    }
  
    monthlySummary.value = data.map(entry => ({
      date: entry.date,
      studentID: entry.studentID,
      firstName: entry.students.firstName,
      lastName: entry.students.lastName,
      attendClass: entry.attendClass,
      missClass: entry.missClass,
      sickLeaveClass: entry.sickLeaveClass,
      personalLeave: entry.personalLeave,
    }))
  
    filterBySearch()
  }
  
  const filterBySearch = () => {
    const query = searchQuery.value.toLowerCase()
  
    filteredWeeklySummary.value = Object.keys(weeklySummary.value)
      .filter(studentId => {
        const student = weeklySummary.value[studentId]
        return student.firstName.toLowerCase().includes(query) || student.lastName.toLowerCase().includes(query)
      })
      .reduce((filtered, studentId) => {
        filtered[studentId] = weeklySummary.value[studentId]
        return filtered
      }, {})
  
    filteredMonthlySummary.value = monthlySummary.value.filter(entry =>
      entry.firstName.toLowerCase().includes(query) || entry.lastName.toLowerCase().includes(query)
    )
  }
  
  const fetchAttendanceSummary = () => {
    if (summaryType.value === 'weekly') {
      fetchWeeklySummary()
    } else if (summaryType.value === 'monthly') {
      fetchMonthlySummary()
    }
  }
  import { useRouter } from 'vue-router'

const router = useRouter()

const goToAttendance = () => {
  router.push('/attendance') // URL ของหน้าการเข้าเรียน
}

  
  fetchAttendanceSummary()
  </script>
  
  <style scoped>
  .attendance-container {
    max-width: 100%;
    padding: 20px;
    background: linear-gradient(135deg, #f0f4ff, #e0eaff);
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }
  
  h2 {
    font-size: 26px;
    text-align: center;
    color: #333;
    margin-bottom: 20px;
    font-weight: bold;
  }
  
  .form-group {
    margin-bottom: 20px;
  }
  
  select,
  input {
    width: 100%;
    padding: 10px;
    font-size: 16px;
    border: 2px solid #d0d8f0;
    border-radius: 8px;
    background-color: #fff;
    transition: border-color 0.3s ease;
  }
  
  select:hover,
  input:hover {
    border-color: #a0b0e0;
  }
  
  .table-responsive {
    overflow-x: auto;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }
  
  th,
  td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
    font-size: 14px;
  }
  
  th {
    background-color: #a0b0e0;
    color: white;
  }
  
  td {
    background-color: #f9f9ff;
  }
  
  @media screen and (max-width: 768px) {
    h2 {
      font-size: 22px;
    }
  
    th,
    td {
      padding: 10px;
      font-size: 12px;
    }
  }
  .navigation-buttons {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  color: white;
  background-color: #4a90e2;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #357abd;
}

/* ปรับแต่งสำหรับหน้าจอมือถือ */
@media screen and (max-width: 768px) {
  button {
    padding: 12px;
    font-size: 14px;
    width: 100%; /* ปุ่มเต็มความกว้าง */
    max-width: 300px; /* จำกัดความกว้างสูงสุด */
  }

  .navigation-buttons {
    margin: 10px 0; /* ลดระยะห่างในมือถือ */
  }
}

  </style>
  