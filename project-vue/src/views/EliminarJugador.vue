<template>
  <div class="eliminar-jugador">
    <h1 class="text-center mt-4 w-100">
       <i class="bi bi-person-x m-3"></i> Eliminar Jugador
    </h1>
    <hr class="w-75 mb-auto" />
    <alert-exito :mensaje="msg" v-if="showAlertaExito" @cerrarExito="cerrarAlertaExito" id="alertaExito" class="mt-5"/>
    <alert-warning :mensaje="msg" v-if="showAlertaWarning" @cerrarWarning="cerrarAlertaWarning" id="alertaWarning" class="mt-5"/>

    <form class="mb-auto mt-5">
      <fieldset class="border border-1 rounded p-5 pt-0 d-flex flex-wrap flex-column">
       <p class="d-flex mt-5">
         <desplegable-equipos :escudos="escudos" :equipos="equipos" @seleccionEquipo="seleccionarEquipo"/>
         <input class="text-center fs-5 form-control" type="text" name="jornada" readonly :value="equipo"/>
       </p>

       <p class="d-flex">
         <desplegable-jugadores :escudos="escudos" :jugadores="jugadores" @seleccionJugador="seleccionarJugador" :isDisabled="isDisabled"/>
         <input class="text-center fs-5 form-control" type="text" name="jornada" readonly :value="jugador.name"/>
       </p>

       <button type="button" :class="desactivarBoton" @click="eliminarJugador">Eliminar</button>
      </fieldset>
    </form>

    <div class="hueco w-100"></div>
  </div>
</template>

<script>
import DesplegableEquipos from "@/components/DesplegableEquipos.vue";
import DesplegableJugadores from "@/components/DesplegableJugadores.vue";
import AlertExito from "@/components/AlertExito.vue";
import AlertWarning from "@/components/AlertWarning.vue";
import axios from "axios";

export default {
    name: "EliminarJugador",
    props: ["escudos"],
    data() {
      return {
        equipos: [],
        jugadores: [],
        equipo: "",
        jugador: "",
        showAlertaExito: false,
        showAlertaWarning: false,
        msg: ""
      }
    },
    components: {
      DesplegableEquipos,
      DesplegableJugadores,
      AlertExito,
      AlertWarning
    },
    methods: {
      getEquipos(URL) {
        this.isLoading = true;
        axios
          .get(URL)
          .then((response) => (this.equipos = response.data))
          .catch((error) => console.error(error))
          .finally(() => (this.isLoading = false));
      },
      seleccionarEquipo(equipo) {
        this.jugador = "";
        this.equipo = equipo;
        this.getJugadores(equipo);
      },
      seleccionarJugador(jugador) {
        this.jugador = jugador;
      },
      getJugadores(equipo) {
        axios.get("http://localhost:3000/players", {
          params: { team: equipo },
        })
        .then((response) => this.jugadores = response.data)
        .then(() => {
          if(this.jugadores < 1) {
            this.msg = `No hay jugadores en ${equipo}`;
            this.mostrarAlertaWarning();
          }
          
        })
        .catch((err) => console.error(err));
      },
      eliminarJugador() {
        axios
          .delete("http://localhost:3000/players/"+this.jugador.id)
          .then(() => {
            this.jugador = "",
            this.equipo = ""
            this.mostrarAlertaExito();
          });
      },
      mostrarAlertaExito() {
        this.showAlertaExito = true;
        this.msg = "Jugador eliminado con éxito";
        this.goto("#alertaExito");
      },
      cerrarAlertaExito() {
        this.showAlertaExito = false;
      },
      goto(refName) { // Saltar a elemento
       var element = this.$refs[refName];

       window.scrollTo(0, element);
      },
      mostrarAlertaWarning() {
        this.showAlertaWarning = true;
        this.goto("#alertaWarning");
      },
      cerrarAlertaWarning() {
        this.showAlertaWarning = false;
      },
    },
    computed: {
      isDisabled() {
        if(this.jugadores.length  < 1) return true;
        return false;
      },
      desactivarBoton() {
        if(this.equipo === "" || this.jugador === "") return "btn btn-outline-danger disabled btn-lg w-75 m-auto mt-3"
        return "btn btn-outline-danger btn-lg w-75 m-auto mt-3";
      }
    },
    mounted() {
      this.getEquipos("http://localhost:3000/clubs");
    },
}
</script>

<style scoped>
.hueco {
  height: 45rem;
}
</style>