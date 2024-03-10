<!-- src/components/Login.vue -->
<template>
    <div class="d-flex align-center justify-center" style="height: 100vh">
        <v-sheet width="400" class="mx-auto">
            <v-form ref="form" @submit.prevent="login" v-model="valid">
                <!-- Campos de Texto para Usuario y Contraseña -->
                <v-text-field variant="outlined" v-model="usuario" :rules="usuarioRules" label="Nombre de Usuario"
                    required></v-text-field>
                <v-text-field variant="outlined" v-model="contrasena" :rules="contrasenaRules" label="Contraseña"
                    type="password" required></v-text-field>

                <!-- Token (Opcional) -->

                <!-- Esta implementacion se va a realizar posteriormente -->
                <a href="#" class="text-body-2 font-weight-regular">¿Olvidaste tu contraseña?</a>
                <!-- Botón de Iniciar Sesión -->
                <v-btn :disabled="!valid" type="submit" color="primary" block class="mt-2">Iniciar Sesión</v-btn>
            </v-form>

            <!-- Enlace para Registrarse -->
            <div class="mt-2">
                <p class="text-body-2">¿No tienes una cuenta? <router-link to="/register">Regístrate aquí</router-link>
                </p>
            </div>
        </v-sheet>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title class="text-h5">Verificación de Identidad</v-card-title>
                <v-card-text>
                    <ReconocimientoFacial @rostroReconocido="procesarRostroReconocido"
                        @confirmarFoto="confirmarIdentidad" />
                    <!-- Asegúrate de que @confirmarFoto esté correctamente vinculado -->
                </v-card-text>
            </v-card>
        </v-dialog>
    </div>
</template>

<script>
import axios from 'axios';
import Swal from 'sweetalert2';
import ReconocimientoFacial from './Login/ReconocimientoFacial.vue'; // Asegúrate de importar correctamente el componente

export default {
    name: 'UserLogin',
    components: {
        ReconocimientoFacial
    },
    data() {
        return {
            valid: false,
            dialog: false,
            usuario: '',
            contrasena: '',
            // token: '',
            usuarioRules: [
                v => !!v || 'El nombre de usuario es requerido',
            ],
            contrasenaRules: [
                v => !!v || 'La contraseña es requerida',
            ],
            mostrarOpcionesFoto: false,
            formDataRostro: null,
            // tokenRules: [
            //     v => !!v || 'El token es requerido',
            // ],
        };
    },
    methods: {
        async login() {

            if (this.$refs.form.validate()) {
                // Si todas las validaciones son correctas, proceder con el inicio de sesión

                try {
                    const response = await axios.post(`${process.env.VUE_APP_AUTH_BASE_URL}/login`, {
                        usuario: this.usuario,
                        contrasena: this.contrasena,
                        // token: this.token
                    });

                    // Verifica que el userId está presente en la respuesta
                    console.log('Respuesta del servidor:', response.data);

                    if (response.data.userId) {
                        // localStorage.setItem('accessToken', response.data.token);
                        localStorage.setItem('userId', response.data.userId);
                        localStorage.setItem('faceId', response.data.faceId);
                        localStorage.setItem('usuarioDescifrado', response.data.usuarioDescifrado);
                        // localStorage.setItem('user', JSON.stringify(response.data.user));
                        this.dialog = true;
                        console.log('UserId almacenado:', localStorage.getItem('userId'));
                        console.log('FaceId almacenado:', localStorage.getItem('faceId'));
                        console.log('Usuario almacenado:', localStorage.getItem('usuarioDescifrado'));
                        // // Redirigir al usuario a la ruta del sistema
                        // this.$router.push('/tangle');
                    } else {
                        console.error('UserId no está presente en la respuesta del servidor');
                    }

                } catch (error) {
                    // Manejar el error de credenciales inválidas
                    if (error.response && error.response.status === 401) {
                        Swal.fire({
                            icon: 'error',
                            title: 'Oops...',
                            text: 'Su Contraseña o Usuario son incorrectos',
                        });
                    } else {
                        // Manejar otros errores
                        Swal.fire({
                            icon: 'error',
                            title: 'Error en el servidor',
                            text: 'Error al iniciar sesión en el servidor',
                        });
                    }
                    console.error(error);
                }
            } else {
                // Si no, mostrar un mensaje de error
                Swal.fire({
                    icon: 'error',
                    title: 'Formulario incompleto',
                    text: 'Por favor complete todos los campos requeridos para iniciar sesión.',
                });
            }

        },
        procesarRostroReconocido(formData) {
            console.log('Procesar rostro reconocido invocado');
            this.formDataRostro = formData;
            this.mostrarOpcionesFoto = true;
        },
        confirmarIdentidad() {
            console.log('Confirmar identidad invocado');
            // Este método se invoca cuando se emite `confirmarFoto` desde ReconocimientoFacial
            if (this.formDataRostro) {
                this.enviarImagenFacial(this.formDataRostro);
            } else {
                console.error("No hay datos de imagen facial para enviar.");
                // Podrías querer mostrar un mensaje de error al usuario aquí también.
                Swal.fire({
                    icon: 'error',
                    title: 'Error',
                    text: 'No hay datos de imagen facial para enviar.',
                });
            }
        },
        volverATomarFoto() {
            this.mostrarOpcionesFoto = false;
            this.dialog = false;
            this.$nextTick(() => {
                this.dialog = true;
            });
        },
        async enviarImagenFacial(formData) {
            console.log('Entro a enviarImagenFacial');
            console.log('Enviando imagen facial:', formData);
            try {
                const faceId = localStorage.getItem('faceId'); // Obtiene el faceId almacenado
                const usuarioDescifrado = localStorage.getItem('usuarioDescifrado');
                const userId = localStorage.getItem('userId');
                // const userString = localStorage.getItem('user');
                console.log('FaceId almacenado:', faceId);
                console.log('Usuario almacenado:', usuarioDescifrado);
                // console.log('Usuario almacenado:', userString
                // console.log('Usuario almacenado:', userString);
                formData.append('faceId', faceId);
                formData.append('userId', userId);
                formData.append('usuarioDescifrado', usuarioDescifrado);
                // formData.append('user', userString);

                const response = await axios.post(`${process.env.VUE_APP_AUTH_BASE_URL}/verify-face`, formData, {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                });

                console.log('Respuesta de verificación facial:', response.data);

                if (response.data.message === 'Verificación facial exitosa') {
                    console.log('Verificación facial exitosa');
                    // Autenticación completa, proceder según sea necesario
                    localStorage.setItem('accessToken', response.data.token);
                    localStorage.setItem('userId', response.data.userId);
                    this.$router.push('/tangle');
                } else {
                    console.log('Verificación facial fallida');
                    this.dialog = false;

                    // Manejar verificación facial fallida
                    Swal.fire({
                        icon: 'error',
                        title: 'Verificación Fallida',
                        text: 'No se pudo verificar la identidad facial.',
                    });
                }
            } catch (error) {
                this.dialog = false;
                console.error(error);
                let message = 'Hubo un problema con la verificación facial.';
                if (error.response) {
                    if (error.response.status === 400) {
                        message = 'No se encontraron rostros en la imagen.';
                    } else if (error.response.status === 401) {
                        message = 'No se pudo verificar la identidad facial. Intente de nuevo.';
                    } else if (error.response.status === 404) {
                        message = 'Usuario no encontrado.';
                    } else if (error.response.status === 500) {
                        message = 'Error interno del servidor.';
                    }
                }

                Swal.fire({
                    icon: 'error',
                    title: 'Error en la verificación',
                    text: message,
                });
            }
        }
    },
    forceReload() {
        this.$router.go();
    }
};
</script>

<style scoped>
.video-preview {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.image-preview {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.swal2-container {
    z-index: 1500 !important;
    /* El z-index debe ser mayor que el del modal Vuetify */
}
</style>
```