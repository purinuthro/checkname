<template>
    <div>
checkname
        <ul>
            <li  v-for="student in students" :key="student.id">
                {{ student }}
             </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
import { createClient } from '@supabase/supabase-js'
const supabaseUrl = 'https://qzdpeqxcgetrapxahvpg.supabase.co'
const supabaseKey = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InF6ZHBlcXhjZ2V0cmFweGFodnBnIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Mjk5MDk5NjksImV4cCI6MjA0NTQ4NTk2OX0.0FLFb3vD5_1zC9Q9m21iAeflH9JOIINkYEW6ItiLBVw"
const supabase = createClient(supabaseUrl, supabaseKey)

const students = ref({})

onMounted(async()=>{
    

    await load()
})
async function load() {
    let { data , error } = await supabase
  .from('students')
  .select('*')

  students.value=data!
}

</script>

<style scoped>

</style>