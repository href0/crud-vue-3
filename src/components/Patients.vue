<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>Patients</h1>
        <hr />
        <br /><br />
        <alert v-if="showMessage" :message="message"></alert>
        <button class="btn btn-success btn-sm" type="button" @click="toggleAddPatientModal">
          Add Patient
        </button>
        <br /><br />
        <div v-if="isLoading" class="">
            <div class="spinner-border text-primary" role="status">
              <span class="visually-hidden">Loading...</span>
            </div>
          </div>
        <table v-if="!isLoading" class="table table-hover">
          <thead>
            <tr>
              <th scope="col">No</th>
              <th scope="col">NIK</th>
              <th scope="col">Name</th>
              <th scope="col">Gender</th>
              <th scope="col">Religion</th>
              <th scope="col">Address</th>
              <th scope="col">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(patient, index) in Patients" :key="index">
              <td>{{ index + 1 }}</td>
              <td>{{ patient.nik }}</td>
              <td>{{ patient.name }}</td>
              <td>{{ patient.sex }}</td>
              <td>{{ patient.religion }}</td>
              <td>{{ patient.address }}</td>
              <td>
                <button class="btn btn-warning btn-sm" type="button" @click="toggleEditPatientModal(patient.id)">
                  Edit
                </button>
                <button class="btn btn-danger btn-sm" style="margin-left:10px;" type="button" @click="handleDeletePatient(patient)">
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
  <!-- add new Patient modal -->
  <div ref="addPatientModal" :class="{ show: activeAddPatientModal, 'd-block': activeAddPatientModal }" class="modal fade"
    role="dialog" tabindex="-1">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Add a new patient</h5>
          <button aria-label="Close" class="close" data-dismiss="modal" type="button" @click="toggleAddPatientModal">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <form>
            <!-- NIK -->
            <div class="mb-3">
              <label class="form-label" for="addPatientNik">NIK</label>
              <input id="addPatientNik" v-model="addPatientForm.nik" class="form-control" :class="{ 'is-invalid': formValidation.nik }" placeholder="157101xxxxxxxxxx"
                type="text" />
              <div class="invalid-feedback">
                {{ formValidation.nik }}
              </div>
            </div>
            <!-- NAME -->
            <div class="mb-3">
              <label class="form-label" for="addPatientName">Nama Lengkap</label>
              <input id="addPatientName" v-model="addPatientForm.name" class="form-control" :class="{ 'is-invalid': formValidation.name }" placeholder="Beruang Madu"
                type="text" />
                <div class="invalid-feedback">
                  {{ formValidation.name }}
                </div>
            </div>
            <!-- SEX -->
            <div class="mb-3">
              <label class="form-label" for="addPatientSex">Jenis Kelamin</label>
              <div class="form-check">
                <input v-model="addPatientForm.sex" value="male" class="form-check-input" type="radio" id="male">
                <label class="form-check-label" for="male">
                  Laki-Laki
                </label>
                
              </div>
              <div class="form-check">
                <input v-model="addPatientForm.sex" value="female" class="form-check-input" type="radio" id="female">
                <label class="form-check-label" for="female">
                  Perempuan
                </label>
              </div>
            </div>
            <!-- Phone -->
            <div class="mb-3">
              <label class="form-label" for="addPatientPhone">No Handphone</label>
              <input id="addPatientPhone" v-model="addPatientForm.phone" class="form-control" :class="{ 'is-invalid': formValidation.phone }" placeholder="628211234321"
                type="number" />
                <div class="invalid-feedback">
                  {{ formValidation.phone }}
                </div>
            </div>
            <!-- Religion -->
            <div class="mb-3">
              <label class="form-label" for="addPatientReligion">Agama</label>
              <!-- <input id="addPatientReligion" v-model="addPatientForm.religion" class="form-control" placeholder="Islam"
                type="text" /> -->
                <select v-model="addPatientForm.religion" class="form-select" :class="{ 'is-invalid': formValidation.religion }"  aria-label="Default select example">
                  <!-- <option selected>Open this select menu</option> -->
                  <option disabled value="">Please select one</option>
                  <option v-for="(religion, index) in Religions">{{ religion }}</option>
                </select>
                <div class="invalid-feedback">
                  {{ formValidation.religion }}
                </div>
            </div>
            <!-- Address -->
            <div class="mb-3">
              <label class="form-label" for="addPatientAddress">Alamat</label>
              <input id="addPatientAddress" v-model="addPatientForm.address" class="form-control" :class="{ 'is-invalid': formValidation.phone }"
                placeholder="Jl satu dua tiga" type="text" />
                <div class="invalid-feedback">
                  {{ formValidation.address }}
                </div>
            </div>

            <div class="btn-group" role="group">
              <button v-if="!isLoadingSubmit" class="btn btn-primary" type="button" @click="handleAddSubmit">
                Submit
              </button>
              <button v-if="isLoadingSubmit" class="btn btn-primary" type="button" disabled>
                <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
                Loading...
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
  <div v-if="activeAddPatientModal" class="modal-backdrop fade show"></div>
  <!-- edit Patient modal -->
  <div ref="editPatientModal" :class="{ show: activeEditPatientModal, 'd-block': activeEditPatientModal }"
    class="modal fade" role="dialog" tabindex="-1">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Update</h5>
          <button aria-label="Close" class="close" data-dismiss="modal" type="button" @click="toggleEditPatientModal(null)">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <div v-if="isLoading" class="">
            <div class="spinner-border text-primary" role="status">
              <span class="visually-hidden">Loading...</span>
            </div>
          </div>
          <form v-if="!isLoading">
            <!-- NIK -->
            <div class="mb-3">
              <label class="form-label" for="editPatientNik">NIK</label>
              <input disabled id="editPatientNik" :class="{ 'is-invalid': formValidation.nik }" v-model="editPatientForm.nik" class="form-control" placeholder="157101xxxxxxxxxx"
                type="text" />
                <div class="invalid-feedback">
                {{ formValidation.nik }}
              </div>
            </div>
            <!-- NAME -->
            <div class="mb-3">
              <label class="form-label" for="editPatientName">Nama Lengkap</label>
              <input id="editPatientName" v-model="editPatientForm.name" :class="{ 'is-invalid': formValidation.name }" class="form-control" placeholder="Beruang Madu"
                type="text" />
                <div class="invalid-feedback">
                {{ formValidation.name }}
              </div>
            </div>
            <!-- SEX -->
            <div class="mb-3">
              <label class="form-label" for="editPatientSex">Jenis Kelamin</label>
              <div class="form-check">
                <input v-model="editPatientForm.sex" value="male" class="form-check-input" type="radio" id="male">
                <label class="form-check-label" for="male">
                  Laki-Laki
                </label>
              </div>
              <div class="form-check">
                <input v-model="editPatientForm.sex" value="female" class="form-check-input" type="radio" id="female">
                <label class="form-check-label" for="female">
                  Perempuan
                </label>
              </div>
            </div>
            <!-- Phone -->
            <div class="mb-3">
              <label class="form-label" for="editPatientPhone">No Handphone</label>
              <input id="editPatientPhone" v-model="editPatientForm.phone" :class="{ 'is-invalid': formValidation.phone }" class="form-control" placeholder="628211234321"
                type="number" />
                <div class="invalid-feedback">
                {{ formValidation.phone }}
              </div>
            </div>
            <!-- Religion -->
            <div class="mb-3">
              <label class="form-label" for="editPatientReligion">Agama</label>
              <!-- <input id="editPatientReligion" v-model="editPatientForm.religion" class="form-control" placeholder="Islam"
                type="text" /> -->
              <select v-model="editPatientForm.religion" :class="{ 'is-invalid': formValidation.religion }" class="form-select" aria-label="Default select example">
                <!-- <option selected>Open this select menu</option> -->
                <option v-for="(religion, index) in Religions">{{ religion }}</option>
              </select>
              <div class="invalid-feedback">
                {{ formValidation.religion }}
              </div>
            </div>
            <!-- Address -->
            <div class="mb-3">
              <label class="form-label" for="editPatientAddress">Alamat</label>
              <input id="editPatientAddress" v-model="editPatientForm.address" :class="{ 'is-invalid': formValidation.address }" class="form-control"
                placeholder="Jl satu dua tiga" type="text" />
                <div class="invalid-feedback">
                {{ formValidation.address }}
              </div>
            </div>

            <div class="btn-group" role="group">
              <button v-if="!isLoadingSubmit" class="btn btn-primary" type="button" @click="handleEditSubmit">
                Submit
              </button>
              <button v-if="isLoadingSubmit" class="btn btn-primary" type="button" disabled>
                <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
                Loading...
              </button>
              <!-- <button class="btn btn-danger btn-sm" type="button" @click="handleAddReset">
                Reset
              </button> -->
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
  <div v-if="activeEditPatientModal" class="modal-backdrop fade show"></div>
</template>

<script>
import axios from "axios";
import Alert from "./Alert.vue";

export default {
  data() {
    return {
      activeAddPatientModal: false,
      addPatientForm: {
        // id     : "",
        nik: "",
        name: "",
        sex: "male",
        religion: "",
        phone: "",
        address: "",
      },
      activeEditPatientModal: false,
      editPatientForm: {
        id       : '',
        nik      : "",
        name     : "",
        sex      : "",
        religion : "",
        phone    : "",
        address  : "",
      },
      Patients: [],
      Religions : ['Islam', 'Kristen', 'Katolik', 'Hindu', 'Buddha', 'Khonghucu'],
      message: "",
      showMessage: false,
      isLoadingSubmit : false,
      isLoading : false,
      formValidation : {
        nik      : "",
        name     : "",
        sex      : "",
        religion : "",
        phone    : "",
        address  : "",
      }
    };
  },
  components: {
    alert: Alert,
  },
  methods: {
    addPatient(payload) {
      this.isLoadingSubmit = true
      const path = "http://localhost:8000/patients";
      axios
        .post(path, payload)
        .then(() => {
          this.isLoadingSubmit = false
          this.toggleAddPatientModal();
          this.getPatients();
          this.initForm();
          this.message = "Patient added";
          this.showMessage = true;
        })
        .catch((error) => {
          console.log(error);
          this.isLoadingSubmit = false
          const dataErr = error.response.data.result
          for(const err of dataErr) {
            this.formValidation[err.field] = err.message
          }
          // this.getPatients();
        });
    },
    getPatients() {
      this.isLoading = true
      const path = "http://localhost:8000/patients";
      axios
        .get(path)
        .then((res) => {
          this.isLoading = false
          this.Patients = res.data.result;
          console.info('patients', this.Patients)
        })
        .catch((error) => {
          this.isLoading = false
          console.error(error);
        });
    },
    getPatientById(patientId) {
      this.isLoading = true
      const path = `http://localhost:8000/patients/${patientId}`;
      axios
        .get(path)
        .then((res) => {
          this.isLoading = false
          this.editPatientForm = res.data.result;
          console.info('patients', this.editPatientForm)
        })
        .catch((error) => {
          this.isLoading = false
          console.error(error);
        });
    },
    handleAddReset() {
      this.initForm();
    },
    handleAddSubmit() {
      this.addPatient(this.addPatientForm);
    },
    initForm() {
      this.addPatientForm.nik = "";
      this.addPatientForm.name = "";
      this.addPatientForm.sex = "";
      this.addPatientForm.address = "";
      this.addPatientForm.religion = "";
      this.addPatientForm.phone = "";

      this.editPatientForm.id = "";
      this.editPatientForm.nik = "";
      this.editPatientForm.name = "";
      this.editPatientForm.sex = "";
      this.editPatientForm.address = "";
      this.editPatientForm.religion = "";
      this.editPatientForm.phone = "";
    },
    initValidation() {
      this.formValidation.nik = "";
      this.formValidation.name = "";
      this.formValidation.sex = "";
      this.formValidation.address = "";
      this.formValidation.religion = "";
      this.formValidation.phone = "";
    },
    toggleAddPatientModal() {
      this.initValidation()
      const body = document.querySelector("body");
      this.activeAddPatientModal = !this.activeAddPatientModal;
      if (this.activeAddPatientModal) {
        body.classList.add("modal-open");
      } else {
        body.classList.remove("modal-open");
      }
    },
    toggleEditPatientModal(patientId) {
      this.initValidation()
      if(patientId) {
        this.initForm()
        this.getPatientById(patientId)
      }
      const body = document.querySelector("body");
      this.activeEditPatientModal = !this.activeEditPatientModal;
      if (this.activeEditPatientModal) {
        body.classList.add("modal-open");
      } else {
        body.classList.remove("modal-open");
      }
      
    },
    handleEditSubmit() {
      // this.toggleEditPatientModal(null);
      this.updatePatient(this.editPatientForm, this.editPatientForm.id);
    },
    updatePatient(payload, patiendId) {
      this.isLoadingSubmit = true
      const path = `http://localhost:8000/patients/${patiendId}`;
      axios.put(path, payload)
        .then(() => {
          this.isLoadingSubmit = false
          this.toggleEditPatientModal(null)
          this.getPatients();
          this.message = 'Patient updated!';
          this.showMessage = true;
        })
        .catch((error) => {
          console.error(error);
          this.isLoadingSubmit = false
          const dataErr = error.response.data.result
          for(const err of dataErr) {
            this.formValidation[err.field] = err.message
          }
          // this.getPatients();
        });
    },
    handleEditCancel() {
      this.toggleEditPatientModal(null);
      this.initForm();
      this.getPatients();
    },
    handleDeletePatient(Patient) {
      this.removePatient(Patient.id);
    },
    removePatient(patiendId) {
      const path = `http://localhost:8000/patients/${patiendId}`;
      axios.delete(path)
        .then(() => {
          this.getPatients();
          this.message = 'Patient removed!';
          this.showMessage = true;
        })
        .catch((error) => {
          console.error(error);
          this.getPatients();
        });
    },
  },
  created() {
    this.getPatients();
  },
};
</script>
