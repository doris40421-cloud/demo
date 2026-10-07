<template>
  <div class="q-pa-xl">
    <!-- 按鈕區塊 -->
    <div class="row q-col-gutter-xs">
      <div class="col-12 col-md-2">
        <q-btn label="SEARCH EMPLOYEES" class="full-width" @click="search" />
      </div>
      <div class="col-12 col-md-2">
        <q-btn label="SAVE EMPLOYEES" class="full-width" @click="openCreateDialog" />
      </div>
      <div class="col-12 col-md-2 offset-md-4">
        <q-btn label="UPDATE EMPLOYEES" class="full-width"
          @click="updateOpenDialog" v-show="selected.length != 0" />
      </div>
      <div class="col-12 col-md-2">
        <q-btn label="DELETE EMPLOYEES" class="full-width" color="red"
          @click="deleteDialog = true" v-show="selected.length != 0" />
      </div>
    </div>
    <q-separator spaced />

    <!-- 輸入區塊 -->
    <div class="row q-col-gutter-xs">
      <div class="col-12 col-md-6">
        <q-input label="First Name" filled clearable
          v-model="request.getEmployees.params.firstName">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Last Name" filled clearable
          v-model="request.getEmployees.params.lastName">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Email" filled clearable type="email"
          v-model="request.getEmployees.params.email">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Phone Number" filled clearable
          v-model="request.getEmployees.params.phoneNumber">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Hire Date From" filled clearable type="date" stack-label
          v-model="request.getEmployees.params.hireDateFrom">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Hire Date To" filled clearable type="date" stack-label
          v-model="request.getEmployees.params.hireDateTo">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Salary From" filled clearable type="number"
          v-model="request.getEmployees.params.salaryFrom">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-input label="Salary To" filled clearable type="number"
          v-model="request.getEmployees.params.salaryTo">
        </q-input>
      </div>
      <div class="col-12 col-md-6">
        <q-select label="Job Title" filled clearable :options="jobs" map-options emit-value
          v-model="request.getEmployees.params.jobId">
        </q-select>
      </div>
      <div class="col-12 col-md-6">
        <q-select label="Department Names" filled clearable :options="departments"
          map-options emit-value multiple
          v-model="request.getEmployees.params.departmentIds">
        </q-select>
      </div>
    </div>
    <q-separator spaced />

    <!-- 顯示區塊 -->
    <div class="row">
      <div class="col">
        <q-table
          title="Employee"
          :rows="employees"
          :columns="columns"
          row-key="employeeId"
          selection="single"
          v-model:selected="selected"
          v-model:pagination="pagination"
          :rows-per-page-options="[5, 10, 20, 50]"
          @request="alterPagination"
        >
          <template v-slot:body-cell-jobTitle="props">
            <q-td :props="props">
              {{ getLabel('jobs', props.row.jobId) }}
            </q-td>
          </template>

          <template v-slot:body-cell-departmentName="props">
            <q-td :props="props">
              {{ getLabel('departments', props.row.departmentId) }}
            </q-td>
          </template>
        </q-table>
      </div>
    </div>
    <!-- 新增修改視窗 -->
    <q-dialog v-model="editDialog" persistent>
      <q-card style="width: 700px; max-width: 80vw;">
        <!-- 標題 -->
        <q-card-section>
          <span class="text-h6">Edit Employee Information</span>
        </q-card-section>

        <!-- 輸入欄位 -->
        <q-card-section>
          <q-input
            v-model="request.saveEmployee.data.employeeId"
            label="Employee ID"
            disable
            v-show="request.saveEmployee.data.employeeId"
          />
          <q-input
            v-model="request.saveEmployee.data.firstName"
            label="First Name"
            clearable
          />
          <q-input
            v-model="request.saveEmployee.data.lastName"
            label="Last Name"
            clearable
          />
          <q-input
            v-model="request.saveEmployee.data.email"
            label="Email"
            clearable
          />
          <q-input
            v-model="request.saveEmployee.data.phoneNumber"
            label="Phone Number"
            clearable
          />
          <q-select
            v-model="request.saveEmployee.data.jobId"
            label="Job Title"
            :options="jobs"
            emit-value
            map-options
            clearable
          />
          <q-select
            v-model="request.saveEmployee.data.departmentId"
            label="Department Name"
            :options="departments"
            emit-value
            map-options
            clearable
          />
          <q-input
            v-model="request.saveEmployee.data.hireDate"
            label="Hire Date"
            type="date"
            stack-label
            clearable
          />
          <q-input
            v-model="request.saveEmployee.data.salary"
            label="Salary"
            type="number"
            clearable
          />
        </q-card-section>

        <!-- 功能按鈕 -->
        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="red" @click="cancelEdit" />
          <q-btn flat label="Confirm" color="primary" @click="confirmEdit" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- 刪除確認視窗 -->
    <q-dialog v-model="deleteDialog" persistent>
      <q-card style="width: 400px; max-width: 80vw;">
        <q-card-section>
          <span class="text-h6">Delete Employee</span>
        </q-card-section>

        <q-card-section v-if="selected.length != 0">
          確定要刪除這位員工嗎？
          <div>Employee ID: {{ selected[0].employeeId }}</div>
          <div>Name: {{ selected[0].firstName }} {{ selected[0].lastName }}</div>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="primary" @click="deleteDialog = false" />
          <q-btn flat label="Delete" color="red" @click="confirmDelete" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>
<script>
import axios from 'axios'

export default {
  data () {
    return {
      columns: [
        { name: 'employeeId', label: 'Employee ID', align: 'left', field: row => row.employeeId },
        { name: 'firstName', label: 'First Name', align: 'left', field: row => row.firstName },
        { name: 'lastName', label: 'Last Name', align: 'left', field: row => row.lastName },
        { name: 'email', label: 'Email', align: 'left', field: row => row.email },
        { name: 'phoneNumber', label: 'Phone Number', align: 'left', field: row => row.phoneNumber },
        { name: 'salary', label: 'Salary', align: 'left', field: row => row.salary },
        { name: 'hireDate', label: 'Hire Date', align: 'left', field: row => row.hireDate },
        { name: 'jobTitle', label: 'Job Title', align: 'left', field: row => row.jobId },
        { name: 'departmentName', label: 'Department Name', align: 'left', field: row => row.departmentId }
      ],
      employees: [],
      jobs: [],
      departments: [],
      selected: [],
      editDialog: false,
      deleteDialog: false,
      pagination: {
        page: 1,
        rowsPerPage: 20,
        rowsNumber: 0
      },
      request: {
        getEmployees: {
          method: 'GET',
          url: '/api/employees',
          params: {
            sort: 'employeeId,asc',
            firstName: null,
            lastName: null,
            email: null,
            phoneNumber: null,
            hireDateFrom: null,
            hireDateTo: null,
            salaryFrom: null,
            salaryTo: null,
            jobId: null,
            departmentIds: []
          },
          // 陣列轉成 departmentIds=10&departmentIds=20 的格式；空值不送出
          paramsSerializer: {
            serialize: (params) => {
              const search = new URLSearchParams()
              Object.keys(params).forEach(key => {
                const value = params[key]
                if (Array.isArray(value)) {
                  value.forEach(v => search.append(key, v))
                } else if (value !== null && value !== undefined && value !== '') {
                  search.append(key, value)
                }
              })
              return search.toString()
            }
          }
        },
        saveEmployee: {
          method: 'POST',
          url: '/api/employee/save',
          data: {}
        },
        deleteEmployee: {
          method: 'POST',
          url: '/api/employee/delete',
          data: {}
        }
      }
    }
  },
  created () {
    // 載入下拉選單的選項
    this.callGetJobs()
    this.callGetDepartments()
  },
  methods: {
    async callGetEmployees () {
      // 向後端要資料
      const response = await axios(this.request.getEmployees)
      // 更新員工資料
      this.employees = response.data.content
      // 更新分頁資訊
      this.pagination.rowsNumber = response.data.totalElements
      this.pagination.rowsPerPage = response.data.size
      this.pagination.page = response.data.number + 1
    },
    search () {
      // 按搜尋時，回到第 1 頁
      this.request.getEmployees.params.page = 0
      this.request.getEmployees.params.size = this.pagination.rowsPerPage
      this.callGetEmployees()
    },
    alterPagination (args) {
      // 按換頁或改筆數時，改寫請求參數（q-table 頁碼從 1 開始，後端從 0 開始）
      this.request.getEmployees.params.page = args.pagination.page - 1
      this.request.getEmployees.params.size = args.pagination.rowsPerPage
      this.callGetEmployees()
    },
    async callGetJobs () {
      const response = await axios.get('/api/jobsLabelValues')
      this.jobs = response.data
    },
    async callGetDepartments () {
      const response = await axios.get('/api/departmentsLabelValues')
      this.departments = response.data
    },
    getLabel (source, id) {
      // 沒有代號就顯示空字串；找不到對應名稱就顯示原本的代號，避免畫面報錯
      if (!id) return ''
      const found = this[source].find(each => each.value.toString() === id.toString())
      return found ? found.label : id
    },
    openCreateDialog () {
      // 新增：表單清空後打開視窗
      this.request.saveEmployee.data = {}
      this.editDialog = true
    },
    updateOpenDialog () {
      // 修改：打開視窗，並把選到的那一列複製一份放進表單
      this.editDialog = true
      this.request.saveEmployee.data = Object.assign({}, this.selected[0])
    },
    cancelEdit () {
      // 取消：關閉視窗並清空表單
      this.editDialog = false
      this.request.saveEmployee.data = {}
    },
    async confirmEdit () {
      // 確認：關閉視窗並送出資料
      this.editDialog = false
      await this.callSaveEmployee()
    },
    async callSaveEmployee () {
      try {
        const response = await axios(this.request.saveEmployee)
        alert(`Employee ID: ${response.data.employeeId}\nName: ${response.data.firstName} ${response.data.lastName}\nhas been saved successfully!`)
        // 存檔後清掉選取，並重新查詢，讓表格更新
        this.selected = []
        await this.callGetEmployees()
      } catch (error) {
        const detail = error.response ? JSON.stringify(error.response.data) : error.message
        alert('Save Employee Failed\n' + detail)
        console.log(error)
      }
      this.request.saveEmployee.data = {}
    },
    async confirmDelete () {
      // 確認刪除：關閉確認視窗並送出刪除
      this.deleteDialog = false
      await this.callDeleteEmployee()
    },
    async callDeleteEmployee () {
      this.request.deleteEmployee.data = { employeeId: this.selected[0].employeeId }
      try {
        await axios(this.request.deleteEmployee)
        alert(`Employee ID: ${this.request.deleteEmployee.data.employeeId}\nhas been deleted.`)
        // 刪除後清掉選取，並重新查詢，讓表格更新
        this.selected = []
        await this.callGetEmployees()
      } catch (error) {
        alert('Delete Failed')
        console.log(error)
      }
    }
  }
}
</script>

<style>
</style>