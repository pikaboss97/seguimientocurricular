<template>
  <div class="row d-flex justify-content-center">
    <div v-if="assignaments.length > 0" class="card col-sm-8 text-center">
      <div class="card-body">
        <h5 class="card-title">Escuela Profesional: {{ studentData.EscuelaProfesional }}</h5>
        <h4 class="card-title">Alumno: {{ studentData.Alumno }}</h4>
        <div class="pgbar">
          <ProgressBar :bgcolor="'#6a1b9a'" :completed=enrolledCredits ></ProgressBar>
        </div>
        <div>
          <section :id="'sec_' + semester.codigo" v-for="semester in SEMESTERS_SECTIONS" :key="semester.codigo">
            <div class="row no-gutters">
              <div class="col-12">
                <p class="title-casino" :class="'Ftitle' + semester.codigo">
                  {{ semester.name }}
                </p>
              </div>
              <div class="circle-container">
                <div v-for="(item, index) in semester.cursos" :key="index">
                  <div v-if="item.matriculado" class="enrolled_curse">
                    <span class="circle-text">{{ item.nombre }}<br>{{ item.codigo }}</span>
                  </div>
                  <div v-if="!item.aprobado && item.nota && !item.matriculado" class="failed_curse">
                    <span class="circle-text">{{ item.nombre }}<br>{{ item.codigo }}</span>
                  </div>
                  <div v-if="item.aprobado" class="aproved_curse">
                    <span class="circle-text">{{ item.nombre }}<br>{{ item.codigo }}</span>
                  </div>
                  <div v-if="item && !item.matriculado && !item.aprobado && !item.nota" class="circle">
                    <span class="circle-text">{{ item.nombre }}<br>{{ item.codigo }}</span>
                  </div>
                </div>
              </div>
            </div>
          </section>
        </div>
      </div>
      <div class="card-footer text-muted">Ponderado: </div>
    </div>
    <div v-if="assignaments.length == 0 && !userNotFound" class="card col-sm-7 text-center">
      <div class="card-body">
        <div class="mb-3 text-center">
          <div class="spinner-border text-success" role="status">
            <span class="visually-hidden">Loading...</span>
          </div>
        </div>
      </div>
      <div class="card-footer text-muted">Ponderado: </div>
    </div>
    <div v-if="assignaments.length == 0 && userNotFound" class="card col-sm-7 text-center">
      <div class="card-body">
        <div class="mb-3 text-center">
          <div class="spinner-border text-success" role="status">
            <span class="visually-hidden">
              Usuario no registrado!!!!
            </span>
          </div>
        </div>
      </div>
      <div class="card-footer text-muted">Pongase en contacto con el administrador... </div>
    </div>
  </div>
</template>

<script>

import axios from 'axios';
import ProgressBar from './ProgressBar.vue';



const SEMESTERS_SECTIONS = [
  { code: 'I', name: "PRIMER SEMESTRE", cursos: [], semester: 1 },
  { code: 'II', name: "SEGUNDO SEMESTRE", cursos: [], semester: 2 },
  { code: 'III', name: "TERCER SEMESTRE", cursos: [], semester: 3 },
  { code: 'IV', name: "CUARTO SEMESTRE", cursos: [], semester: 4 },
  { code: 'V', name: "QUINTO SEMESTRE", cursos: [], semester: 5 },
  { code: 'VI', name: "SEXTO SEMESTRE", cursos: [], semester: 6 },
  { code: 'VII', name: "SEPTIMO SEMESTRE", cursos: [], semester: 7 },
  { code: 'VIII', name: "OCTAVO SEMESTRE", cursos: [], semester: 8 },
  { code: 'IX', name: "NOVENO SEMESTRE", cursos: [], semester: 9 },
  { code: 'X', name: "DECIMO SEMESTRE", cursos: [], semester: 10 },
];

export default {
  name: 'record-card',
  props: {
    code: String
  },
  components: {
    ProgressBar,
  },
  data() {
    return {
      SEMESTERS_SECTIONS,
      studentData: {},
      assignaments: [],
      electives: [],
      totalCredits: 215,
      approvedCredits: 0,
      enrolledCredits: 0,
      banners: 0,
      userNotFound:false
    }
  },
  methods: {
    async fetchData() {
      const user = this.code;
      const year = user.slice(2,6);
      const credentials = {
        username: user,
      };
      try {
        this.cleanConstSections();
        const record = await axios.get('https://pdfapi-a7a4.onrender.com/curricula?ep=FIIS&v='+year);
        this.setRecordToList(record.data)

        const response = await axios.post('https://pdfapi-a7a4.onrender.com/notas', credentials, { 'Content-Type': 'application/json' });
        if (response.data) {
          this.totalCredits = response.data.TC;
          this.studentData = response.data
          this.assignaments = response.data.Asignaturas;
          this.progressValues(response.data.CA, response.data.CM);
          this.setSubjectsStudied();
          this.setElectivesStudied();
        } else {
          this.userNotFound = true;
          console.log("error", response);
        }

      } catch (error) {
        console.log(error);
      }
    },
    progressValues(ca, cm) {
      this.approvedCredits = ((ca / this.totalCredits) * 100).toFixed(2);
      this.enrolledCredits = (((ca + cm) / this.totalCredits) * 100).toFixed(2);
    },
    setRecordToList(record) {
      const values = Object.values(record.data);
      this.SEMESTERS_SECTIONS.map(e => {
        values.forEach(value => {
          if (e.semester == value.semestre) {
            e.cursos.push(value)
          }
        });
      })
    },
    setSubjectsStudied() {
      this.SEMESTERS_SECTIONS.map(e => {
        e.cursos.map(f => {
          this.assignaments.forEach(value => {
            if (f.codigo == value.codigo) {
              f.matriculado = value.matriculado;
              f.aprobado = value.aprobado;
              f.nota = value.nota;
            }
          });
        })

      })
    },
    setElectivesStudied() {
      this.electives = this.assignaments.filter(curso => { return curso.electivo });
      this.SEMESTERS_SECTIONS.map(e => {
        e.cursos.map(f => {
          this.electives.forEach((value, index) => {
            if (f.class == 'E' + (index + 1)) {
              f.aprobado = true;
            }
          });
        })

      })
    },
    cleanConstSections() {
      this.SEMESTERS_SECTIONS.map(e => {
        e.cursos = [];
      })
    },
    isMobile() {
      let mobile =
        /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
          navigator.userAgent
        );
      return mobile;
    },
    showAlert() {
      this.$swal.fire({
        title: 'Resumen',
        imageWidth: 400,
        imageHeight: 200,
        imageAlt: 'Custom image',
      })
    }

  },
  async mounted() {
    this.banners = this.isMobile() ? 1 : 3;
    await this.fetchData();
  },
}
</script>

<style scoped>
.asg-body {
  margin: 2rem;
}

.circle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #ffffff;
  color: rgb(7, 7, 7);
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
  border: 1px solid;
}

.aproved_curse {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #57bc66;
  color: rgb(7, 7, 7);
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
}

.failed_curse {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #de7b7b;
  color: rgb(7, 7, 7);
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
}

.enrolled_curse {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #2196F3;
  color: rgb(7, 7, 7);
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
}

.circle:hover,
.failed_curse:hover,
.enrolled_curse:hover,
.aproved_curse:hover {
  background-color: #de7a17;
}

.circle-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
  margin: 5px;

}

.pgbar {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 10px;
}


.carousel_container {
  margin-left: 2px;
  margin-right: 2px;
}

.carousel__item {
  margin-right: 15px;
  margin-left: 15px;
  min-height: 50px;
  width: 100%;
}

.carousel_img {
  max-width: 100%;
  max-height: 100%;
}

.initial_card {
  height: 100px;
  border: 2px solid;
  border-color: #ffffff;
}

.js .inputfile {
  width: 0.1px;
  height: 0.1px;
  opacity: 0;
  overflow: hidden;
  position: absolute;
  z-index: -1;
}

.inputfile-4+label figure {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background-color: #d3394c;
  display: block;
  padding: 20px;
  margin: 0 auto 10px;
}

/* Estilos para pantallas pequeñas */
@media (max-width: 768px) {
  .circle {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background-color: #ffffff;
    color: rgb(7, 7, 7);
    font-weight: bold;
    font-size: 7px;
    cursor: pointer;
    border: 1px solid;
    border-color: rgba(41, 40, 40, 0.675);
  }

  .aproved_curse {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background-color: #57bc66;
    color: rgb(7, 7, 7);
    font-weight: bold;
    font-size: 7px;
    cursor: pointer;
  }

  .failed_curse {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background-color: #de7b7b;
    color: rgb(7, 7, 7);
    font-weight: bold;
    font-size: 7px;
    cursor: pointer;
  }

  .enrolled_curse {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background-color: #2196F3;
    color: rgb(7, 7, 7);
    font-weight: bold;
    font-size: 7px;
    cursor: pointer;
  }

  .carousel_container {
    margin-top: 0px;
    margin-left: 0px;
    margin-right: 0px;
  }

  .carousel__item {
    margin: 0px;
    min-height: 100px;
    min-width: 100px;
    border-radius: 8px;

  }

  .carousel_img {
    max-width: 100%;
    max-height: 100%;
  }
}
</style>
