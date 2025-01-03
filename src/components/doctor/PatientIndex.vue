<template>
  <div class="patient-index">
    <h1 class="mb-4">家属列表</h1>
    <el-input
      v-model="search"
      placeholder="搜索家属"
      class="mb-4"
    ></el-input>
    <el-table :data="filteredPatients" style="width: 100%">
      <el-table-column prop="displayId" label="ID" width="80" align="center"></el-table-column>
      <el-table-column prop="user.username" label="用户名" width="250"></el-table-column>
      <el-table-column label="状态" width="100">
        <template #default="scope">
          {{ scope.row.user.status === 'active' ? '活跃' : '停用' }}
        </template>
      </el-table-column>
      <el-table-column label="头像" width="100">
        <template #default="scope">
          <el-avatar :size="40" :src="scope.row.user.avatarUrl" />
        </template>
      </el-table-column>
      <el-table-column label="操作" min-width="180">
        <template #default="scope">
          <el-button size="small" @click="viewPatientDetails(scope.row)">查看详情</el-button>
          <el-button size="small" type="primary" @click="startConsultation(scope.row)">开始咨询</el-button>
        </template>
      </el-table-column>
      <template #empty>
        <el-empty description="没有家属"></el-empty>
      </template>
    </el-table>

    <el-dialog v-model="showDetails" title="家属详情" width="55%">
      <PatientDetails :patient="selectedPatient" @close="showDetails = false" />
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import axios from 'axios'
import { ElMessage } from 'element-plus'
import PatientDetails from './PatientDetails.vue'

const router = useRouter()
const store = useStore()

const patients = ref([])
const search = ref('')
const showDetails = ref(false)
const selectedPatient = ref(null)

const fetchPatients = async () => {
  try {
    const response = await axios.get('/api/api/doctor/relation/selectMyPatientsAndRelationId', {
      headers: {
        Authorization: `Bearer ${store.state.token}`
      }
    });

    // 处理响应数据，替换 avatarUrl
    patients.value = response.data.map(patient => {
      const newAvatarUrl = patient.user.avatarUrl.replace('http://localhost:8080/UserAvatar/', 'https://zeropw.cn:8081/UserAvatar/');
      return {
        ...patient,
        user: {
          ...patient.user,
          avatarUrl: newAvatarUrl
        }
      };
    });
  } catch (error) {
    console.error('Failed to fetch patients:', error);
    ElMessage.error('获取家属列表失败');
  }
};

const filteredPatients = computed(() => {
  return patients.value
      .filter(patient =>
          patient.user.username.toLowerCase().includes(search.value.toLowerCase()) ||
          patient.user.userId.includes(search.value)
      )
      .map((patient, index) => ({
        ...patient,
        displayId: index + 1
      }))
})

const viewPatientDetails = (patient) => {
  selectedPatient.value = patient
  showDetails.value = true
}

const startConsultation = (patient) => {
  router.push({ path: '/consultation', query: { relationId: patient.relationId } })
}

onMounted(() => {
  fetchPatients()
})
</script>

<style scoped>
.patient-index {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}
</style>