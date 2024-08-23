<template>
  <MenuPrincipal />
  <div class="recuperacion-container">
    <div class="recuperacion-box" v-if="!tokenValido">
      <h2 class="recuperacion-title">Recuperar Contraseña</h2>
      <b-form @submit.prevent="solicitarRecuperacion">
        <b-form-group
          label="Correo Electrónico:"
          label-for="email-input"
          class="recuperacion-label"
        >
          <b-form-input
            id="email-input"
            v-model="email"
            type="email"
            required
            placeholder="Ingrese su correo electrónico"
            class="recuperacion-input"
          ></b-form-input>
        </b-form-group>

        <b-button
          type="submit"
          class="recuperacion-button"
          :disabled="cargando"
        >
          Enviar
        </b-button>
      </b-form>
    </div>

    <div class="recuperacion-box" v-if="tokenValido">
      <h2 class="recuperacion-title">Restablecer Contraseña</h2>
      <b-form @submit.prevent="restablecerContrasena">
        <b-form-group
          label="Nueva Contraseña:"
          label-for="nueva-contrasena-input"
          class="recuperacion-label"
        >
          <b-form-input
            id="nueva-contrasena-input"
            v-model="nuevaContrasena"
            type="password"
            required
            placeholder="Ingrese su nueva contraseña"
            class="recuperacion-input"
          ></b-form-input>
        </b-form-group>

        <b-form-group
          label="Confirmar Nueva Contraseña:"
          label-for="confirmar-contrasena-input"
          class="recuperacion-label"
        >
          <b-form-input
            id="confirmar-contrasena-input"
            v-model="confirmarContrasena"
            type="password"
            required
            placeholder="Confirme su nueva contraseña"
            class="recuperacion-input"
          ></b-form-input>
        </b-form-group>

        <b-button
          type="submit"
          class="recuperacion-button"
          :disabled="cargando"
        >
          Restablecer Contraseña
        </b-button>
      </b-form>
    </div>
  </div>
</template>

<script>
import MenuPrincipal from "@/components/MenuPrincipal.vue";
import axios from "axios";
import Swal from "sweetalert2";

export default {
  components: {
    MenuPrincipal,
  },
  data() {
    return {
      email: "",
      nuevaContrasena: "",
      confirmarContrasena: "",
      tokenValido: false,
      token: "",
      cargando: false, // Indicador de carga
    };
  },
  created() {
    const urlParams = new URLSearchParams(window.location.search);
    this.token = urlParams.get("token");
    if (this.token) {
      this.verificarToken();
    }
  },
  methods: {
    async solicitarRecuperacion() {
      this.cargando = true; // Inicia la carga
      try {
        const response = await axios.post(
          "https://back-wwpy.onrender.com/api/solicitar-recuperacion",
          { email: this.email }
        );
        Swal.fire({
          icon: "success",
          title: "Correo enviado",
          text: response.data.message,
          confirmButtonColor: "#0a641a",
        });
      } catch (error) {
        Swal.fire({
          icon: "error",
          title: "Error",
          text: error.response?.data?.error || "Ocurrió un error inesperado.",
          confirmButtonColor: "#0a641a",
        });
      } finally {
        this.cargando = false; // Finaliza la carga
      }
    },
    async verificarToken() {
      this.cargando = true; // Inicia la carga
      try {
        const response = await axios.post(
          "https://back-wwpy.onrender.com/api/verificar-token",
          { token: this.token }
        );
        if (response.data.tokenValido) {
          this.tokenValido = true;
        } else {
          Swal.fire({
            icon: "error",
            title: "Error",
            text:
              response.data.message || "El token no es válido o ha expirado.",
            confirmButtonColor: "#0a641a",
          });
        }
      } catch (error) {
        Swal.fire({
          icon: "error",
          title: "Error",
          text: error.response?.data?.message || "Error al verificar el token.",
          confirmButtonColor: "#0a641a",
        });
      } finally {
        this.cargando = false; // Finaliza la carga
      }
    },
    async restablecerContrasena() {
      if (this.nuevaContrasena !== this.confirmarContrasena) {
        Swal.fire({
          icon: "error",
          title: "Error",
          text: "Las contraseñas no coinciden.",
          confirmButtonColor: "#0a641a",
        });
        return;
      }

      this.cargando = true; // Inicia la carga
      try {
        const response = await axios.post(
          "https://back-wwpy.onrender.com/api/reset-password",
          {
            token: this.token,
            nuevaContrasena: this.nuevaContrasena,
          }
        );
        Swal.fire({
          icon: "success",
          title: "Contraseña restablecida",
          text: response.data.message,
          confirmButtonColor: "#0a641a",
        }).then(() => {
          this.$router.push("/inicio-sesion");
        });
      } catch (error) {
        Swal.fire({
          icon: "error",
          title: "Error",
          text:
            error.response?.data?.error || "Error al restablecer la contraseña",
          confirmButtonColor: "#0a641a",
        });
      } finally {
        this.cargando = false; // Finaliza la carga
      }
    },
  },
};
</script>

<style scoped>
.recuperacion-container {
  background-color: #ffffff;
  min-height: 80vh;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
  padding: 0 20px;
  box-sizing: border-box;
}

.recuperacion-box {
  background-color: #0a641a;
  padding: 20px;
  border-radius: 15px;
  width: 100%;
  max-width: 500px;
  box-shadow: 0 30px 15px rgba(0, 0, 0, 0.2);
  text-align: center;
}

.recuperacion-title {
  font-weight: bold;
  color: #ffffff;
  margin-top: 15px;
  margin-bottom: 15px;
}

.recuperacion-label {
  font-weight: bold;
  margin-top: 20px;
  color: #ffffff;
  text-align: left;
}

.recuperacion-input {
  border-radius: 5px;
}

.recuperacion-button {
  background-color: #ff9800;
  color: #ffffff;
  font-weight: bold;
  border: none;
  width: 100%;
  border-radius: 10px;
  margin-top: 15px;
  padding: 10px;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

.recuperacion-button:hover {
  background-color: #e68900;
}
</style>
