<!-- /src/components/Login/ReconocimientoFacial.vue -->
<template>
    <div class="camera-wrapper">
        <!-- Solo muestra el video o el canvas, no ambos al mismo tiempo -->
        <video ref="video" autoplay v-show="!imagenCapturada" class="video-preview"></video>
        <canvas ref="canvas" v-show="imagenCapturada" class="image-preview"></canvas>

        <!-- Botones para la lógica de captura -->
        <div class="button-group">
            <v-btn color="primary" @click="capturarFoto" v-show="!imagenCapturada">Capturar Foto</v-btn>
            <v-btn color="error" @click="volverATomar" v-show="imagenCapturada">Volver a Tomar</v-btn>
            <v-btn color="success" @click="confirmarFoto" v-show="imagenCapturada">Confirmar Identidad</v-btn>
        </div>
    </div>
</template>

<style scoped>
.camera-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.video-preview,
.image-preview {
    max-width: 100%;
    max-height: calc(100vh - 120px);
    /* Ajusta este valor según el espacio que quieras dejar para los botones */
    border-radius: 4px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.button-group {
    display: flex;
    justify-content: center;
    gap: 10px;
    /* Espacio entre botones */
    margin-top: 20px;
}
</style>

<script>
export default {
    data() {
        return {
            imagenCapturada: false,
        };
    },
    mounted() {
        this.iniciarCamara();
    },
    methods: {
        iniciarCamara() {
            navigator.mediaDevices.getUserMedia({ video: true })
                .then(stream => {
                    this.$refs.video.srcObject = stream;
                })
                .catch(error => {
                    console.error("Error al acceder a la cámara:", error);
                });
        },
        capturarFoto() {
            this.$nextTick(() => {
                const canvas = this.$refs.canvas;
                const video = this.$refs.video;
                if (canvas && video) {
                    // Ajusta el tamaño del canvas para que coincida con el video
                    canvas.width = video.videoWidth;
                    canvas.height = video.videoHeight;

                    const context = canvas.getContext('2d');
                    context.drawImage(video, 0, 0, video.videoWidth, video.videoHeight);
                    video.srcObject.getTracks().forEach(track => track.stop());
                    this.imagenCapturada = true;
                    canvas.toBlob(blob => {
                        const formData = new FormData();
                        formData.append('faceImage', blob, 'faceImage.png');
                        this.$emit('rostroReconocido', formData);
                    }, 'image/png');
                }
            });
        },
        volverATomar() {
            this.imagenCapturada = false;
            this.iniciarCamara();
        },
        confirmarFoto() {
            this.$emit('confirmarFoto');
            console.log('Evento confirmarFoto emitido');
        }
    }
};
</script>