<template>
  <div class="container-fluid py-4">
    
    <a-button 
     v-if="currentUser.role == 'system_admin'"
      danger
      class="mb-2" 
      type="primary"  
      @click="showModal"
      >Create Subject
    </a-button>

     <a-table
      bordered
      rowKey="id"
      :columns="columns"
      :data-source="data"
      :scroll="{ x: 600 }"
      :pagination="{ pageSize: 10 }"
      >
        <!-- <template
          v-if="currentUser.role == 'system_admin'"
          #operation="{ record }"
          >
          <a-button
            type="primary"
            style="background: greenyellow; color:black"
            @click.prevent="editUser(record.id)"
            shape="round"
            size="small"
            >Edit</a-button
          >
          {{ " " }}
          <a-button
            type="primary"
            danger
            @click.prevent="deleteSubject(record.id)"
            shape="round"
            size="small"
            >Delete</a-button
          >
        </template> -->
     </a-table>
    <a-modal v-model:visible="visible" title="Create Subject" centered @ok="createSubject">
     <a-spin :spinning="loading" size="large">
      <div class="mb-2">
        <h6 class="p-0">Select College </h6>
         <select
            v-model="college_id"
            @change="onChangeSelectCollege"
            class="form-select p-2 bg-white border"
            style="font-size: 17px"
            required
          >
            <option
              style="font-size: 18px"
              v-for="option in collegeList"
              :value="option.id"
              v-bind:key="option.id"
            >
              {{ option.code.toUpperCase() }}
            </option>
          </select>
      </div>
      <div class="mb-2">
        <h6 class="p-0">Select Department </h6>
          <select
            v-model="department_id"
            @change="onChangeSelect1"
            class="form-select p-2 bg-white border"
            style="font-size: 17px"
            >
            <option
              style="font-size: 18px"
              v-for="option in departmentList"
              :value="option.id"
              v-bind:key="option.id"
              >
              {{ option.code.toUpperCase() }} 
            </option>
          </select>
      </div>
      <!-- <div class="mb-2">
        <h6 class="p-0">Select Curriculum </h6>
         <select
            v-model="curriculum_id"
            @change="onChangeSelect1"
            class="form-select p-2 bg-white border"
            style="font-size: 17px"
            required
          >
            <option
              style="font-size: 18px"
              v-for="option in currList"
              :value="option.id"
              v-bind:key="option.id"
            >
              {{ option.code.toUpperCase() }} ({{option.major_code}})
            </option>
          </select>
      </div> -->
       <div class="mb-2">
        <h6 class="p-0">Code</h6>
          <a-input
            size="large"
            placeholder="Enter Curriculum Code"
            v-model:value="code"
            required
          />
      </div>
      <div class="mb-2">
        <h6 class="p-0">Description</h6>
        <a-input
          size="large"
          placeholder="Enter Curriculum Description"
          v-model:value="description"
          required
          />
      </div>
      <div class="mb-2">
        <h6 class="p-0">Units</h6>
        <a-input
          size="large"
          type="number"
          placeholder="Enter Units"
          v-model:value="units"
          required
          />
      </div>
     </a-spin>
    </a-modal>
  
  </div>
</template>

<script>

import { mapState } from "vuex";

export default {
  name: "subjects",
  data(){
    return{
    data: [],
    code: "",
    loading: false,
    visible: false,
    description: "",
    units: null,
    college_id: null,
    college_code: "",
    department_id: null,
    curriculum_id: null,
    curriculum_code: "",
    departmentList: [],
    collegeList: [],
    currList: [],
    columns: [
        {
          title: 'Code',
          width: 90,
          customRender: (s) => s.record.code.toUpperCase() ,
        },
        {
          title: 'Description',
          dataIndex: 'description',
          width: 200,
        },
        {
          title: 'Units',
          dataIndex: 'units',
          width: 100,
        },
        // {
        //   title: 'Year',
        //   width: 100,
        //   customRender: (s) => ordinal_suffix_of(s.record.year) + " year",
        // },
        //  {
        //   title: 'Semester',
        //   width: 100,
        //   customRender: (s) => ordinal_suffix_of(s.record.semester) + " semester",
        // },
        {
          title: 'College',
          width: 100,
          customRender: (s) => s.record.college_code.toUpperCase() ,
        },
       {
          title: 'Department',
          customRender: (s) => <div>{s.record.department_description} ({s.record.department_code.toUpperCase()})</div> ,
          width: 200,
        },
      ],
    }
  },

  computed: {
    ...mapState(["currentUser"]),
  },
  mounted: function (){
    this.loading = true;
    this.$secured.get('/api/v1/subjects')
        .then(response => { this.data = response.data})
        .catch(error => { console.log(error) });

        if (this.currentUser.role == "system_admin") {
      const additionCol = {
        title: "Operation",
        dataIndex: "operation",
        width: 155,
        fixed: "right",
        slots: {
          customRender: "operation",
        },
      };
      this.columns.push(additionCol);
      this.$secured.get("/api/v1/roles")
        .then((response) => {
          this.roleSelection = response.data.data;
          this.role = response.data.data[0];
          this.filter = [...this.filter, ...this.roleSelection];
        })
        .catch((error) => {
          console.log(error);
        });
    }
    setTimeout(() => {
      this.loading = false;
    }, 700)  
  },
  
  methods: {
    showModal: function(){
      this.visible = true
      this.fetchCollege()
    },
    fetchCollege() {
      this.$secured
        .get("/api/v1/colleges")
        .then((response) => {
          this.collegeList = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    fetchDepartment(){
       this.$secured.get("/api/v1/get_department_by_college_id?college_id="+ this.college_id)
        .then(response => { this.departmentList = response.data})
        .catch(error => { console.log(error) })
    },
    fetchCurr() {
      this.$secured
        .get("/api/v1/get_curriculum_by_college_id?college_id="+ this.college_id)
        .then((response) => {
          this.currList = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    onChangeSelectCollege(event) {
      this.college_id = event.target.value;
      this.fetchDepartment()
    },
    onChangeSelect1(event) {
      this.department_id = event.target.value;
    },
    createSubject(){
      this.loading = true
      this.$secured.post("/api/v1/subjects", {subject: {
        code: this.code.toLowerCase(),
        description: this.description,
        units: this.units,
        college_id: this.college_id,
        department_id: this.department_id
      }})
      .then(response=>{
        this.$toast.open({
            message: "Succesfully Created",
            type: "success",
            position: "top-right",
            duration: 2500,
          })
        console.log(response.data)
        this.visible = false
        this.fetchCurr()
      })
      .catch(error => {
        this.$toast.open({
            message: "Something is Wrong",
            type: "warning",
            position: "top-right",
            duration: 2500,
          });
          this.loading = false
          console.log(error.response.data.code)
      })
      this.code = ""
      this.description = ""
      this.units = null
      this.college_id = null
      this.department_id = null
      this.loading = false
      this.visible = false
    },
    deleteSubject(id) {
      console.log(id)
      this.loading = true
      this.$secured.delete(`/api/v1/subjects/${id}`)
      this.loading = false
    },
    editUser(id) {
      console.log(id);
    },
  }
  // mounted: function (){
  //  this.fetchCurr()
  // },
};
</script>
