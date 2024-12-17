<template>
  <div class="container">
    <h2>เลือกชั้นเรียนและห้อง</h2>
    <button class="btn navigate-btn" @click="goToAttendanceSummary">
  ดูสรุปการเช็คชื่อ
</button>

    <div class="select-container">
      <select v-model="selectedGrade" class="dropdown" @change="loadClassrooms">
        <option disabled value="">เลือกชั้นเรียน</option>
        <option v-for="grade in grades" :key="grade" :value="grade">
          {{ grade }}
        </option>
      </select>

      <select v-model="selectedClassroom" class="dropdown" @change="loadStudents">
        <option disabled value="">เลือกห้องเรียน</option>
        <option v-for="classroom in classrooms" :key="classroom" :value="classroom">
          ห้อง {{ classroom }}
        </option>
      </select>
    </div>

    <div v-if="students.length > 0" class="table-container">
      <h3>ตารางเช็คสถานะชั้นม.{{ selectedGrade }},ห้อง{{ selectedClassroom }}</h3>
      <table class="custom-table">
        <thead>
          <tr>
            <th>เลขที่</th>
            <th>ชื่อจริง</th>
            <th>นามสกุล</th>
            <th>สถานะ</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="student in students" :key="student.id">
            <td>{{ student.number }}</td>
            <td>{{ student.firstName }}</td>
            <td>{{ student.lastName }}</td>
            <td>
              <button
                v-for="(btn, index) in attendanceButtons"
                :key="index"
                :class="['btn', btn.class]"
                @click="markAttendance(student.id, btn.status)"
                :disabled="hasAttendanceRecord(student.id)"
              >
                {{ btn.label }}
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useNuxtApp } from '#app'

const { $supabase } = useNuxtApp()
const grades = ['1', '2', '3', '4', '5', '6']
const classrooms = ref([])
const selectedGrade = ref('')
const selectedClassroom = ref('')
const students = ref([])
const attendanceRecords = ref([])

const attendanceButtons = [
  { label: 'Attend', status: 'attend', class: 'success' },
  { label: 'Miss', status: 'miss', class: 'danger' },
  { label: 'Sick', status: 'sick', class: 'warning' },
  { label: 'Personal Leave', status: 'personal', class: 'info' }
]

const loadClassrooms = () => {
  if (selectedGrade.value) {
    classrooms.value = [1, 2, 3, 4, 5] // Classroom 1-5
  }
}

const loadStudents = async () => {
  if (selectedGrade.value && selectedClassroom.value) {
    const { data, error } = await $supabase
      .from('students')
      .select('*')
      .eq('grade', selectedGrade.value)
      .eq('classroom', selectedClassroom.value)

    if (error) {
      console.error('Error fetching students:', error)
    } else {
      students.value = data
      await loadAttendanceRecords() // Load attendance records for the selected grade and classroom
    }
  }
}

// Load attendance records for today's date and selected classroom
const loadAttendanceRecords = async () => {
  const date = new Date().toISOString().split('T')[0]
  const { data, error } = await $supabase
    .from('attendance')
    .select('*')
    .eq('date', date)
    .in('studentID', students.value.map(student => student.id))

  if (error) {
    console.error('Error fetching attendance records:', error)
  } else {
    attendanceRecords.value = data
  }
}

// Check if a student already has an attendance record for today
const hasAttendanceRecord = (studentId) => {
  return attendanceRecords.value.some(record => record.studentID === studentId)
}

const markAttendance = async (studentId, status) => {
  const date = new Date().toISOString().split('T')[0]

  // Check if a record already exists for this student
  if (hasAttendanceRecord(studentId)) {
    alert('This student already has an attendance status for today.')
    return
  }

  const attendanceData = {
    date: date,
    studentID: studentId,
    attendClass: status === 'attend' ? 1 : 0,
    missClass: status === 'miss' ? 1 : 0,
    sickLeaveClass: status === 'sick' ? 1 : 0,
    personalLeave: status === 'personal' ? 1 : 0,
  }

  const { error } = await $supabase.from('attendance').insert([attendanceData])

  if (error) {
    console.error('Error inserting attendance:', error)
  } else {
    alert('Attendance marked successfully!')
    // Reload attendance records to disable buttons for students who have an entry
    await loadAttendanceRecords()
  }
}
const goToAttendanceSummary = () => {
  window.location.href = '/attendance-summary'; // ใช้ลิงก์ของหน้า `attendance-summary`
}
</script>

<style scoped>
body {
  background: linear-gradient(135deg, #84fab0 0%, #8fd3f4 100%);
  font-family: 'Arial', sans-serif;
  min-height: 100vh;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  max-width: 900px;
  margin: 20px auto;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.select-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 20px;
}

.dropdown {
  padding: 10px;
  font-size: 16px;
  width: 100%;
  border-radius: 8px;
  border: 1px solid #ccc;
}

.table-container {
  margin-top: 30px;
}

.custom-table {
  width: 100%;
  border-collapse: collapse;
}

.custom-table th,
.custom-table td {
  padding: 12px;
  text-align: left;
  border: 1px solid #ddd;
}

.custom-table th {
  background-color: #f0f0f0;
}

.btn {
  padding: 8px 12px;
  margin-right: 5px;
  border: none;
  cursor: pointer;
  font-size: 14px;
  border-radius: 5px;
}

.btn.success {
  background-color: #4caf50;
  color: white;
}

.btn.danger {
  background-color: #f44336;
  color: white;
}

.btn.warning {
  background-color: #ff9800;
  color: white;
}

.btn.info {
  background-color: #2196f3;
  color: white;
}

@media screen and (max-width: 768px) {
  .container {
    padding: 15px;
    max-width: 100%;
  }
  
  .table-container {
    overflow-x: auto;
  }

  .custom-table th,
  .custom-table td {
    padding: 10px;
    font-size: 14px;
  }

  .btn {
    padding: 6px 8px;
    font-size: 12px;
    margin-right: 3px;
  }
}

@media screen and (max-width: 600px) {
  .dropdown {
    font-size: 14px;
  }

  .btn {
    padding: 5px 7px;
    font-size: 12px;
  }
}
.navigate-btn {
  background-color: #49a1ff;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 20px;
  font-size: 16px;
}

.navigate-btn:hover {
  background-color: #053970;
}


</style>
